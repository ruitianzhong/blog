---
title: "Notes on KVM API"

date: 2024-01-29T09:48:00+08:00

lastmod: 2024-01-29T09:48:00+08:00

summary: Essential KVM API

subtitle: Essential KVM API

draft: false

featured: false

authors:
  - admin

categories:
  - Linux
  - Virtualization
---


## Simple KVM Snippet

Based on [kvmtest.c](https://lwn.net/Articles/658512/)

```c

#include <fcntl.h>
#include <linux/kvm.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/types.h>
void error(const char *err) {
  printf("error\n");
  perror(err);
  exit(EXIT_FAILURE);
}

int main(void) {
  int kvm, vmfd, vcpufd, ret;

  const uint8_t code[] = {
      0xba, 0xf8, 0x03, /* mov $0x3f8, %dx */
      0x00, 0xd8,       /* add %bl, %al */
      0x04, '0',        /* add $'0', %al */
      0xee,             /* out %al, (%dx) */
      0xb0, '\n',       /* mov $'\n', %al */
      0xee,             /* out %al, (%dx) */
      0xf4,             /* hlt */
  };

  uint8_t *mem;
  struct kvm_sregs sregs;
  size_t mmap_size;
  struct kvm_run *run;
  kvm = open("/dev/kvm", O_RDWR | O_CLOEXEC);
  if (kvm == -1) {
    error("open");
  }
  ret = ioctl(kvm, KVM_GET_API_VERSION, NULL);
  if (ret == -1) {
    error("GET_API_VERSION");
  }
  if (ret != 12) {
    error("Unexpected api version");
  }
  vmfd = ioctl(kvm, KVM_CREATE_VM, (unsigned long)0);
  if (vmfd == -1) {
    error("CREATE_VM");
  }
  mem = mmap(NULL, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS,
             -1, 0);
  if (!mem)
    error("mmap");
  memcpy(mem, code, sizeof(code));
  struct kvm_userspace_memory_region region = {
      .slot = 0,
      .guest_phys_addr = 0x1000,
      .memory_size = 0x1000,
      .userspace_addr = (uint64_t)mem,
  };
  ret = ioctl(vmfd, KVM_SET_USER_MEMORY_REGION, &region);
  if (ret == -1)
    error("KVM_SET_USER_MEMORY_REGION");
  vcpufd = ioctl(vmfd, KVM_CREATE_VCPU, (unsigned long)0);
  if (vcpufd == -1)
    error("KVM_CREATE_VCPU");
  ret = ioctl(kvm, KVM_GET_VCPU_MMAP_SIZE, NULL);
  if (ret == -1)
    error("KVM_GET_VCPU_MMAP_SIZE");
  mmap_size = ret;
  if (mmap_size < sizeof(*run)) {
    error("unexpectedly small mmap_size");
  }
  run = mmap(NULL, mmap_size, PROT_READ | PROT_WRITE, MAP_SHARED, vcpufd, 0);
  if (!run) {
    error("mmap vcpu");
  }
  ret = ioctl(vcpufd, KVM_GET_SREGS, &sregs);
  if (ret == -1) {
    error("KVM_GET_SREGS");
  }
  sregs.cs.base = 0;
  sregs.cs.selector = 0;
  ret = ioctl(vcpufd, KVM_SET_SREGS, &sregs);
  if (ret == -1)
    error("KVM_SET_SREGS");
  struct kvm_regs regs = {
      .rip = 0x1000,
      .rax = 2,
      .rbx = 2,
      .rflags = 0x2,
  };

  ret = ioctl(vcpufd, KVM_SET_REGS, &regs);
  if (ret == -1) {
    error("KVM_SET_REGS");
  }

  while (1) {
    ret = ioctl(vcpufd, KVM_RUN, NULL);
    if (ret == -1) {
      error("KVM_RUN");
    }
    switch (run->exit_reason) {
    case KVM_EXIT_HLT:
      puts("Guest host halt");
      return 0;
    case KVM_EXIT_IO:
      if (run->io.direction == KVM_EXIT_IO_OUT && run->io.size == 1 &&
          run->io.port == 0x3f8 && run->io.count == 1) {
        putchar(*(((char *)run) + run->io.data_offset));

      } else
        error("unhandled KVM_EXIT_IO");
      break;
    case KVM_EXIT_FAIL_ENTRY:
      error("KVM_EXIT_FAIL_ENTRY");
    case KVM_EXIT_INTERNAL_ERROR:
      error("INTERNAL ERROR");
    default:
      error("placeholder");
    }
  }
}
```

