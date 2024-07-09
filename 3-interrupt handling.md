- interrupt process
```
while True:
    # exec current instruction
    read EIP's code instruction to instruction buffer;
    increase the EIP;
    execute the introcution in instruction buffer;

    #exam interrupts from interrupt request register
    check for intrrupts()
    if there is:
      save current state;
      interrupt = get highest prio interrupt;
      handle interrupt(interrup)
      restore saved state;

    # move to next instruction

```

- Does interrupt can be interrupted by another interrupt? 
```

  yes,
  1. NMI interrupt what ever can be interrupt the ongoing interrupt as it has high priority (The NMI is assigned an interrupt number of 2),
     even it's disabled. (which means none-NMI interrupt非可屏蔽中断)
  2. when irq handling is not disabling the irq explictly. can be interrupted by higher priority interrupt.
  and Whatever irq disable or not, the new irq will buffered  in interrupt queue of interrupt controller (ordered by priority,time...)
  so that it can interrupt CPU when CPU is enabled the interrupt, or if the new is NMI interrupt which can'be blocked to execute by irq disable.
  Note: interrupt controller is responsible for receiving signal of interrupt, 
        re-arrange them based on priority, then save them, until CPU is able to handle it.
```

- local_irq_save（flag） the param is value passing, not reference.
```
As it is not function, value was pushed into stack.
but it is macro, so the flag is updated by flags = arch_local_irq_save();
```

- local_irq_save and local_irq_restore 
```
local_irq_save (flag) shall be used when inner/ongoing function doesn't know previous irq status(enable or disable),
will have other disable irq + action + restore irq.
This makes sense, because interrupts are disabled at different levels. 

local_irq_save(flags1);    /* interrupts are now disabled */ saving the previous state in their one unsigned long flags argument.
/* ... */

// this below call maybe hidden inside the sub-function, which shall use local_irq_save(flags) which shall not impact later on restore the already interrupt disabled state!!!
local_irq_save(flags2);    // disable interrupts, and save the previous state
local_irq_restore(flags2); // restore the previous state: in this case. it's disabled. as before local_irq_save(flag2), local_irq_save(flags1) is executed.
/* ... */
local_irq_restore(flags1); /* interrupts are restored to their previous state, which is enabled probably. as it's state before the execution of local_irq_save(flags1); 


#define arch_local_irq_save arch_local_irq_save
static inline unsigned long arch_local_irq_save(void)
{
	unsigned long flags;

	asm volatile(
		"	mrs	%0, " IRQMASK_REG_NAME_R "	@ arch_local_irq_save\n"-----------------mrs means move value of the system register to common register's.
		"	cpsid	i"-------------------------------------------------------disable all maskable interrupt.
		: "=r" (flags) : : "memory", "cc");
	return flags;
}

/*
 * restore saved IRQ & FIQ state
 */
#define arch_local_irq_restore arch_local_irq_restore
static inline void arch_local_irq_restore(unsigned long flags)
{
	asm volatile(
		"	msr	" IRQMASK_REG_NAME_W ", %0	@ local_irq_restore"--------------msr means move value of common register to system register.
		:
		: "r" (flags)
		: "memory", "cc");
}



Interrupt Enabling
Code may need to disable interrupts, do something (critical region), then re-enable interrupts
but if interrupts were already disabled, re-enabling them might be incorrect. so that is shall use local_irq_save+local_irq_restore.
use:
local_irq_save (flags);

... /* critical region */

local_irq_restore (flags);


If more than one function in the call chain might need to disable interrupts, local_irq_save should be used.


#define raw_local_irq_save(flags)			\
	do {						\
		typecheck(unsigned long, flags);	\
		flags = arch_local_irq_save();		\
	} while (0)
#define raw_local_irq_restore(flags)	
```




```
