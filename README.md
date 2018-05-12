# FileSystem
Linux FileSystem knowledge collecting

# BIO
BIO(Block Input Output)
## Page cache与预读：
在Linux中，内存充当硬盘的page cache，所以，每次读的时候，会先check你读的那一部分硬盘文件数据是否在内存命中，如果没有命中，才会去硬盘；如果已经命中了，就直接从内存里面读出来。如果是写的话，应用如果是以非SYNC方式写的话，写的数据也只是进内存，然后由内核帮忙在适当的时机writeback进硬盘。
内存到硬盘的转换：
Bio （(include/linux/blk_types.) ：在Linux里面，用于描述硬盘里面要真实操作的位置与page cache的页映射关系的数据结构是bio。
BIO和page cache是一对多的关系。
它是一个描述硬盘里面的位置与page cache的页对应关系的数据结构，每个bio对应的硬盘里面一块连续的位置，每一块硬盘里面连续的位置，可能对应着page cache的多页，或者一页，所以它里面会有一个bio_vec *bi_io_vec的表。
对于连续的硬盘存储单元：
	Linux会为这一次4页的读，分配1个bio就足够了，并且让这个bio里面分配4个bi_io_vec，指向4个不同的内存页。
对于不连续的硬盘存储单元：
	Linux会为这一次4页的读，分配4个bio，并且让这4个bio里面，每个分配1个bi_io_vec，指向4个不同的内存页面。
混搭的情况，同上两种方式的结合。
## BIO与request：
Bio -> request -> plug list -> elevator -> dispatch -> 存储设备执行IO
Plug List : 合并bio
在Linux中，每个task_struct（对应一个进程，或轻量级进程——线程），会有一个plug的list
// block/blk-core.c
void blk_queue_bio(struct request_queue *q, struct bio *bio)
把bio合并进入一个进程本地plug list里面的一个request，如果无法合并，则造一个新的request。request里面包含一个bio的list，这个list的bio对应的硬盘位置，最终在硬盘上是连续存放的。
电梯算法 ： 合并request
这个电梯调度，其实目的3个：
  1.	进一步的合并request
  2.	把request对硬盘的访问变得顺序化
  3.	执行QoS (质量服务)
QoS：CFQ调度算法，可以根据进程的ionice，调整不同进程访问硬盘的时候的优先级。IO电梯调度算法有：cfq, noop, deadline。






