# Memory-mapped file

Memory-mapped file, as its name suggest, maps address space in RAM to a file that is accessible and visible via the manipulation of the file system on your Operating System. To understand the usage of the `mmap()` system call, the following illustration will help explains the concept of sharing flag while using `mmap()`:

|         |    File-backed (default)   | Anonymous               |
|---------|----------------------------|-------------------------|
| Private:| Init memory based on file. | Like malloc(), init to 0|
| Shared: | Init memory based on file. Write to file when done. Shared between processes. | Share memory between processes. Init to 0.|

`void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);`

In order to use `mmap()`, we need to know the size of the memory we are allocating and we need to use `open()` to create a file descriptor for `mmap()` to associate memory address space in the physical RAM to an actual file in your file system under the Operating System.

`mmap()` also requires permission specification by setting the `PROT` flag, there are quite a few permission we can set, `PROT_READ`, `PROT_WRITE`, `PROT_EXEC` and `PROT_NONE`. In the end, we can also specify an offset for the contents of a file mapping to begin at certain length inside the file, an offset must be a multiple of the page size as returned by `sysconf(_SC_PAGE_SIZE)`.

```
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>

int main(int argc, char **argv)
{
	int fd, sharing_flags, permission;
	off_t offset = 0;
	size_t s = sizeof(int*8);
	int* addr;

	fd = open(argv[1], O_RDONLY);

	permission = PROT_READ|PROT_WRITE;
	sharing_flags = MAP_PRIVATE;

	addr = (int *) mmap(NULL, s, permission, sharing_flags, fd, offset);

	if(addr == MAP_FAILED)
	{
		perror("Error while allocating address space with mmap()");
		exit(1);
	}

	*(addr[0]) = 234;
	close(fd);

}
```

After we are done with the mapped file, we need to call `munmap()` to disassociate the mapped file from physical RAM, the usage is as follow:

```
{
...
	if(munmap(addr, s) == -1) // We need to perform error validation in order to know if the operation is performed correctly.
	{
		perror("Error while unmapping the mapped file from RAM");
		exit(1);
	}
...
}
```