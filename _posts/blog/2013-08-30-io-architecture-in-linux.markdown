---
layout: post
title: Block IO Architecture in Linux
description: IO设备是Linux中重要的组成部分，其中Block设备又是比较常见的，如硬盘、CDROM等。而且对Block设备的研究也对接下来队Xen的IO优化做好铺垫。
category: blog
tags:
- Block IO
- Linux
---

## Block设备处理的一般流程

  1. 通过系统函数调用相应文件系统函数，传递文件描述符和偏移量。
  2. 文件系统会判断是否所请求的数据已经可用，有时，数据已经存在了RAM里，所以不需要从块设备中重新读取或写入。如果需要发起读取或写入操作，文件系统会处理。
  3. 如果要发起读取或写入操作，就必须先确定数据的物理地址，这需要**mapping layer**，通常需要一下两步：
	- 判断文件系统的块大小（Block Size）,同时确定请求数据的范围，即这些数据的文件块号（file block number）。文件块号即所请求的块在该文件里的块索引号。
	- 映射层（mapping layer）调用文件系统的相关函数去获取文件的inode然后获取请求数据在磁盘上的逻辑块号（logical block number）。逻辑块号即数据所在的块在磁盘上的块索引号。因为数据的块可能是在磁盘上不连续的，所以在inode中有一个数据结构把每一个文件块号和一个逻辑块号建立起了映射。
  4. 内核将读写请求发送给块设备，这里的处理由通用块层（generic block layer）处理。通常用“block io”即bio来表示一个I/O操作。
  5. 在generic block layer之下，是IO调度器（I/O Scheduler）。它会对数据请求进行重新的排序和合并等操作，使得更符合磁盘的物理特性以提高访问效率。
  6. 最后，块设备驱动会把命令发给物理硬件接口完成I/O操作。

## I/O世界中的数据单位

IO操作涉及到内核中自下到上很多层次，所以在不同的层次中用来表示IO数据的单位也不尽相同。

### 扇区（Sector）

扇区是磁盘中的基本数据单位，而磁盘的数据操作一般都是对**连续的**扇区进行的。一般一个扇区的大小是512 bytes，但是有些磁盘也会采用比较大的扇区，如1024bytes或者2048bytes。虽然磁盘的物理结构比较复杂，但是抽象来看，磁盘管理器会把磁盘看成是一个大的扇区数组（array）。

扇区被认为是数据传输的最基本单位，不能传输比一个扇区还要小的数据，但是可以一次传输多个扇区的数据。在Linux中，扇区的大小通常是512bytes，所以存储在块设备上的数据通常用第一个扇区的索引好和包含的扇区数来表示。而扇区索引是存在**sector_t**类型的变量中，这个变量可能是32bits或者64bits的。

### 块（Blocks）

块是文件系统中进行数据传输的基本单位。在Linux中，块大小必须是2的幂次，同时不能大于一个页帧（page frame），而且块大小还必须是扇区大小的倍数。

块不是针对块设备的单位，而是针对文件系统的，一个块设备上可以建立拥有不同块大小的分区。另外，每一个对块设备文件（block device file）直接发起的操作都是一次原始访问（raw access），它们跳过了文件系统，而内核在执行这些访问操作的时候都会采用最大的块大小，即4096 bytes。

每一个块都有其对应的块缓存（block buffer），用来让内核通过内存访问块数据。它相当于块数据的一个缓存，当读的时候可以先读到内存再进行操作，写的时候类似。每一个块缓存都有一个描述符，这个描述符被称为buffer head，是存在一个**buffer_head**的类型的结构体中。描述符中包含了所有内核操作块数据需要的信息。

### 段（Segments）

段的使用首先要提到的DMA操作，DMA操作大致分为两种，一种是传统的方式，每次的DMA操作只能针对一段连续的IO数据；另一种是分散-收集（sactter-gather）DMA，这种DMA操作可以针对一个IO数据链表，而数据链表元素之间不要求是连续的。对于每个scatter-gather DMA，块设备驱动药发送给磁盘控制器一下信息：
	
1. 初始磁盘扇区号和总的扇区数；
2. 包含起始地址和长度的内存区域链表。

在进行scatter-gather DMA的过程中，块设别驱动操作的基本数据单元是段（segment）。一个段可以是一个内存页或者是若干包含连续磁盘扇区的内存页。

在IO调度器中可以对io进行合并，合并后的大io对应的内存区域被称为物理段（phisical segment）。而另一种处理总线地址（bus address）和物理地址（physical address）之间映射的合并后对应的内存区域被称为硬件段（hardware segment）。

## 通用块处理层（Generic Block Layer）

这个层次是对内核块数据处理的抽象，首先，来看一下内核io的表示。

### Bio

bio是io操作的结构体，它存储了一个io操作所需要的信息。其结构为：
	
	sector_t				bi_sector			First sector on disk of block I/O operation
    struct bio *			bi_next				Link to the next bio in the request queue
    struct block_device *	bi_bdev				Pointer to block device descriptor
    unsigned long			bi_flags			Bio status flags
    unsigned long			bi_rw				I/O operation flags
    unsigned short			bi_vcnt				Number of segments in the bio’s bio_vec array
    unsigned short			bi_idx				Current index in the bio’s bio_vec array of segments
    unsigned short			bi_phys_segments	Number of physical segments of the bio after merging
    unsigned short			bi_hw_segments		Number of hardware segments after merging
    unsigned int			bi_size				Bytes (yet) to be transferred
    unsigned int			bi_hw_front_size	Used by the hardware segment merge algorithm
    unsigned int			bi_hw_back_size		Used by the hardware segment merge algorithm
    unsigned int			bi_max_vecs			Maximum allowed number of segments in the bio’s bio_vecarray
    struct bio_vec *		bi_io_vec			Pointer to the bio’s bio_vec array of segments
    bio_end_io_t *			bi_end_io			Method invoked at the end of bio’s I/O operation
    atomic_t				bi_cnt				Reference counter for the bio
	void *					bi_private			Pointer used by the generic block layer and the I/O completion method of the block device driver
    bio_destructor_t *		bi_destructor		Destructor method (usually bio_destructor()) invoked when the bio is being freed

每一个bio主要是用**bio_vec**结构体来表示的，bio中有变量比bi_\io\_vec指向一个bio\_vec链表的第一个元素，同时bi\_vcnt记录了这个数组的元素个数。bio\_vec结构体所包含的变量有：

	struct page *	bv_page		Pointer to the page descriptor of the segment’s page frame
	unsigned int	bv_len		Length of the segment in bytes
	unsigned int	bv_offset	Offset of the segment’s data in the page frame

bio结构中的数据会随着io操作的进行变化，例如bi\_idx，当一个scatter-gather DMA操作无法一次完成一个bio包含的数据时bi\_idx会更新为下一个还未被操作的段。

### 磁盘和分区的表示

这里说的磁盘（disk）是广义上的块设备，可以是硬盘，软盘，或者是CDROM。同时一个磁盘还可以逻辑上分为若干个不同的分区。

磁盘是用结构体gendisk表示的，其包含：

	int									major			Major number of the disk
	int									first_minor		First minor number associated with the disk
	int									minors			Range of minor numbers associated with the disk
	char [32]							disk_name		Conventional name of the disk (usually, the canonical name of the corresponding device file)
	struct hd\_struct **				part			Array of partition descriptors for the disk
	struct block_device_operations *	fops			Pointer to a table of block device methods
	struct request_queue *				queue			Pointer to the request queue of the disk
	void *								private_data	Private data of the block device driver
	sector\_t							capacity		Size of the storage area of the disk (in number of sectors)
	int									flags			Flags describing the kind of disk (see below)
	char [64]							devfs_name		Device filename in the (nowadays deprecated) devfs special filesystem
	int									number			No longer used
	struct device *						driverfs_dev	Pointer to the device object of the disk’s hardware device
	struct								kobject			kobj Embedded kobject
	struct timer_rand_state *			random			Pointer to a data structure that records the timing of the disk’s interrupts; used by the kernel built-in random number generator
	int									policy			Set to 1 if the disk is read-only (write operations forbidden), 0 otherwise
	atomic\_t							sync_io			Counter of sectors written to disk, used only for RAID
	unsigned long						stamp			Timestamp used to determine disk queue usage statistics
	unsigned long						stamp_idle		Same as above
	int									in_flight		Number of ongoing I/O operations
	struct disk\_stats *				dkstats			Statistics about per-CPU disk usage

其中,flags用来表示磁盘的状态：GENHD\_FL\_UP为已初始化并在工作；GENHD\_FL\_REMOVABLE为可移除磁盘。另外，fops指向一个block\_device\_operations的结构体，它存了块设备可以进行的一些操作函数：

	open			Opening the block device file
	release			Closing the last reference to a block device file
	ioctl			Issuing an ioctl() system call on the block device file (uses the big kernel lock)
	compat_ioctl	Issuing an ioctl() system call on the block device file (does not use the big kernel lock)
	media_changed	Checking whether the removable media has been changed (e.g., floppy disk) 
	revalidate_disk Checking whether the block device holds valid data

磁盘所包含的分区存在part变量中，它指向了一个hd\_struct结构体的数组，用来存储当前磁盘上的一些分区信息，hd\_struct包含了：

	sector_t		start_sect		Starting sector of the partition inside the disk
	sector_t		nr_sects		Length of the partition (number of sectors)
	struct kobject	kobj			Embedded kobject (see the section “Kobjects” in Chapter 13)
	unsigned int	reads			Number of read operations issued on the partition
	unsigned int	read_sectors	Number of sectors read from the partition
	unsigned int	writes			Number of write operations issued on the partition
	unsigned int	write_sectors	Number of sectors written into the partition
	int				policy			Set to 1 if the partition is read-only, 0 otherwise
	int				partno			The relative index of the partition inside the disk

### 请求提交

前面说的都是块设备处理中一些常用到的数据结构，下面要说的就是io请求的提交。首先，io请求是以bio的形式存在的，所以要先分配生成bio结构，并且初始化它。接下来，内核会通过generic\_make\_request函数去继续提交过程，其中的流程为：

1. 检测bio->bi\_sector，即io请求的第一个扇区，是否超过块设备的范围；
2. 获取和块设备相关联的请求队列q；
3. 调用block\_wait\_queue\_running函数检测io调度器当前是否正在被替换，如果是，则休眠当前进程直到io调度器重新开启；
4. 调用blk\_partition\_remap函数检测当前的块设备引用是否是分区，如果是的话，需要做以下转换：
	- 更新hd\_struct中的read\_sector，reads，write\_sectors，writes；
	- 调整bio->bi\_sector为分区起点扇区在整个磁盘中的索引号；
	- 设置bio->bi\_bdev为整个磁盘的块设备描述符（bio->bd\_contains）。
	完成了这些之后，以后所有操作就都是针对整个磁盘进行的；
5. 调用函数将bio请求插入到请求队列中；
6. 返回。


