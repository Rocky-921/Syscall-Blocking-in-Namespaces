We have tried to intercept syscalls and apply namespace checking using the following methods:-
1) Hooks (Implementing hooks as loadable module)
2) LSM (Linux Security Module)
3) eBPF (Extended Berkeley Packet Filter) :- see eBPF_block_mkdir
