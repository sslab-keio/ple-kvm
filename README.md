# Mitigating Excessive Pause-Loop Exiting in VM-Agnostic KVM

**These patches are for Linux Kernel 5.6**

In our research work presented last week at the VEE 2021 conference [1], we
found out that a lot of continuous Pause-Loop-Exiting (PLE) events occur
due to three problems we have identified: 1) Linux CFS ignores hints from
KVM; 2) IPI receiver vCPUs in user-mode are not boosted; 3) IPI-receiver
that has halted is always a candidate for boost.  We have intoduced two
mitigations against the problems.

To solve problem (1), patch 1 increases the vruntime of yielded vCPU to
pass the check `if (cfs_rq->next && wakeup_preempt_entity(cfs_rq->next,
left) < 1)` in `struct sched_entity * pick_next_entity()` if the cfs_rq's
skip and next are both vCPUs in the same VM. To keep fairness it does not
prioritize the guest VM which causes PLE, however it improves the
performance by eliminating unnecessary PLE. Also we have confirmed
`yield_to_task_fair` is called only from KVM.

To solve problems (2) and (3), patch 2 monitors IPI communication between
vCPUs and leverages the relationship between vCPUs to select boost
candidates.

Our approach reduces the total number of PLE events by up to 87.6 % in four
8-vCPU VMs in over-subscribed scenario with the Linux kernel 5.6.0. Please
find the patch below.

[1] Kenta Ishiguro, Naoki Yasuno, Pierre-Louis Aublin, and Kenji Kono.
    "Mitigating excessive vCPU spinning in VM-agnostic KVM".
    In Proceedings of the 17th ACM SIGPLAN/SIGOPS International Conference
    on Virtual Execution Environments (VEE 2021).
    Association for Computing Machinery, New York,
    NY, USA, 139--152.  https://dl.acm.org/doi/abs/10.1145/3453933.3454020

