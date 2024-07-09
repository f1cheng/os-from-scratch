
- To understand how context switching is implemented in Assembly Language. Here is AT&T code example
v1:
   ![image](https://github.com/upempty/os-from-scratch/assets/52414719/7973c622-37d2-4bbe-a731-1f8e04ca26c4)
  
   https://medium.com/@iggeehu/speed-running-operating-systems-day-2-learning-assembly-language-by-parsing-context-switch-code-b82dd7561c4

v2:
```
; Source: xv6/swtch.S
; Context switch between kernel routine/stacks
;
;   void switch_kernel_context(struct context **old_context, struct context *new_context);
; 
; Save the current registers on the stack, creating
; a struct context, and save its address in *old.
; Switch stacks to new and pop previously-saved registers.

global switch_kernel_context
switch_kernel_context:

  mov eax, [esp + 4]; struct context **old_context
  mov edx, [esp + 8]; struct context *new_context

  ; Save old callee-saved registers
  push ebp
  push ebx
  push esi
  push edi

  ; Switch stacks
  mov [eax], esp; *old_context = esp, save pointer to old stack to *old
  mov esp, edx; esp = new_context, switch to new stack

  ; Load new callee-saved registers
  pop edi
  pop esi
  pop ebx
  pop ebp
  ret ; pop eip and jmp


typedef struct context {
  uint32_t edi;
  uint32_t esi;
  uint32_t ebx;
  uint32_t ebp;
  uint32_t eip;
} context;
```

v3
```
x86 syntax: dest<-src:

[global switch_to]

switch_to:
    ; 保存现场
    mov eax, [esp + 4]------------old context ptr.
    
    mov [eax + 0],  esp---------------here inside esp, it is stored this current old eip which is pointering next instruction when calling switch_to.
    mov [eax + 4],  ebp
    mov [eax + 8],  ebx
    mov [eax + 12], esi
    mov [eax + 16], edi
    ; 保存标志寄存器
    pushf
    pop ecx
    mov [eax + 20], ecx

    ; 加载新环境
    mov eax, [esp + 8]-------------new context ptr storing into eax.

    mov esp, [eax + 0]-------------inside esp, it is stored eip(fn), then, later "ret" it will pop eip and jmp it, to execute the new context's eip(fn).
<<<<
It is set during process creation phase for esp storage.
    uint32_t *stack_top = (uint32_t *)((uint32_t)new_proc + STACK_SIZE);
    *(--stack_top) = (uint32_t)arg;
    *(--stack_top) = (uint32_t)kthread_exit;
    *(--stack_top) = (uint32_t)fn;
    new_proc->context.esp = (uint32_t)new_proc + STACK_SIZE - 3 * sizeof(uint32_t);
<<<<

    mov ebp, [eax + 4]
    mov ebx, [eax + 8]
    mov esi, [eax + 12]
    mov edi, [eax + 16]
    mov eax, [eax + 20]
    push eax
    popf

    ret
```
