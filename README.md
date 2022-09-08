# Summary - NVMe Emulation Performance Optimization

**GSoC contributor**: *Jinhao Fan*

**Mentors**: *Klaus Jensen*, *Keith Busch*



In this project, I added the following features to QEMU emulated NVMe:

1. Reduced MMIO’s with **shadow doorbell buffer**.
2. Lightweight MMIO with **ioeventfd**.
3. Thread-safe irq delivery with **irqfd**.
4. Dedicated emulation thread with **iothread**.
5. Minimal latency with **polling**.

Features 1, 2, 4 and 5 are planned features that we thought of when writing the project proposal. During development we found Feature 3 necessary for features 4 and 5, so we also implemented that support.

With all these features, we achieved our goal of making QEMU NVMe emulation performance comparable to virtio-blk.

|       **QD**       | **1** | **4** | **16** | **64** |
| :----------------: | :---: | :---: | :----: | :----: |
|      **nvme**      |  53   |  155  |  245   |  309   |
|     virtio-blk     |  59   |  185  |  260   |  256   |
|  **nvme+polling**  |  123  |  165  |  189   |  191   |
| virtio-blk+polling |  88   |  212  |  210   |  213   |

My work is presented at KVM Forum 2022 ([pdf](https://static.sched.com/hosted_files/kvmforum2022/56/NVMe-Jinhao.pdf)). I've been told that these optimizations have been utilized at multiple orginizations.

## Merged patches

1. [hw/nvme: Add shadow doorbell buffer support](https://patchew.org/QEMU/20220616123408.3306055-1-fanjinhao21s@ict.ac.cn/) (3f7fe8d)
2. [hw/nvme: Use ioeventfd to handle doorbell updates](https://patchew.org/QEMU/20220705142403.101539-1-fanjinhao21s@ict.ac.cn/) (2e53b0b)
3. [hw/nvme: Add helper functions for qid-db conversion](https://patchew.org/QEMU/20220803015836.3590335-1-fanjinhao21s@ict.ac.cn/) (387350d)
4. [block/io_uring: add missing include file](https://patchew.org/QEMU/20220721065645.577404-1-fanjinhao21s@ict.ac.cn/) (77e3f03)

## Not yet merged patches

1. [hw/nvme: support irq(de)assertion with eventfd](https://patchew.org/QEMU/20220827091258.3589230-1-fanjinhao21s@ict.ac.cn/20220827091258.3589230-2-fanjinhao21s@ict.ac.cn/)
2. [hw/nvme: use KVM irqfd when available](https://patchew.org/QEMU/20220827091258.3589230-1-fanjinhao21s@ict.ac.cn/20220827091258.3589230-3-fanjinhao21s@ict.ac.cn/)
3. [hw/nvme: add iothread support](https://patchew.org/QEMU/20220827091258.3589230-1-fanjinhao21s@ict.ac.cn/20220827091258.3589230-4-fanjinhao21s@ict.ac.cn/)
4. [hw/nvme: add polling support](https://patchew.org/QEMU/20220827091258.3589230-1-fanjinhao21s@ict.ac.cn/20220827091258.3589230-5-fanjinhao21s@ict.ac.cn/)

## Future work

1. **Get all my outstanding patches merged**.
2. **Add better API for irqfd**. This is an important feature for ensuring thread-safe interrupt delivery. However, we only have low-level APIs with very few documentation. It will be great to add a better wrapper with common functionalities and enough documentation
3. **Measure and understand the performance implications of each optimization**. We saw performance improvements and some surprising regressions along the way. We also know the mechanisms of our optimizations. But we do not understand enough about why some optimization caused certain degree of performance improvements. We need more measurements to understand that.

## Summary

A big thank you to my mentors and other community members! It's a great pleasure to submit patches and get them upstreamed to a well-respected open-source project like QEMU. The experience is great. All the community members are kind, professional and helpful. I learned a lot during this GSoC program, not only on hard-core knowledge about the NVMe protocol, but also soft skills like how to seek help from community members and how to use git properly.

We also faced some problems during the journey. For example, we found the NVMe spec is not clear enough about [shadow doorbell buffer for admin queue](https://lore.kernel.org/qemu-devel/Yqeo4EKtQJq8XRm+@kbusch-mbp.dhcp.thefacebook.com/). We also found an interesting implementation decision in Linux NVMe driver that [creates CQs prior to allocating interrupt lines](https://lore.kernel.org/qemu-devel/YvKJk2dYiwomexFv@kbusch-mbp.dhcp.thefacebook.com/), which makes irqfd hard to use. Some other points are irqfd as a mandatory feature for IOThread support, as well as MSI-X masking/unmasking under irqfd. These problems are unfortunately poorly documented, if not undocumented. Solving the problems requires digging deep into the spec or the underlying code. A lot of time are spent on these problems.

Fortunately, despite these challenges, I managed to accomplish my tasks and delivered the expected result. My optimizations can enable enginners and researcher to emulate fast NVMe devices that are emerging nowadays. The improved performance of emulated NVMe also makes it possible for OSes without virtio drivers to use high-performance virtual block devices, because most OSes have built-in driver for NVMe. I will be happy to come back some day to continue contributions to the QEMU community.
