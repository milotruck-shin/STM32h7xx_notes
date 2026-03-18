

```markdown
# STM32 CMSIS Register Mapping Notes (FDCAN Example)

## Purpose
This note explains how STM32 maps peripheral hardware registers into C code using CMSIS, so we can access registers by name (e.g. `FDCAN1->CCCR`) instead of manually typing raw addresses.

---

## 1) Memory-Mapped I/O (MMIO) in STM32
Peripheral registers are located at fixed memory addresses.  
CMSIS represents them with:

1. **Base address macros**
2. **A C struct** matching register layout
3. **A typed pointer macro** to that struct

This gives readable and safer code. A clever way to map the registers through stuct rather than explicitly writing every macro Def of register values

---

## 2) Address Chain (How `FDCAN1` is formed)

```c
#define FDCAN1_BASE (D2_APB1PERIPH_BASE + 0xA000UL)
#define FDCAN1      ((FDCAN_GlobalTypeDef *) FDCAN1_BASE)
```

- `FDCAN1_BASE`: numeric start address of FDCAN1 register block.
- `FDCAN1`: same address cast to `FDCAN_GlobalTypeDef *`.

Firstly, (FDCAN_GlobalTypeDef *) FDCAN1_BASE means that the pointer variable has a value of FDCAN1_BASE. Equivalent to `FDCAN_GlobalTypeDef *FDCAN1 = FDCAN1_BASE.`
<br>So `FDCAN1` is a typed pointer to the entire FDCAN1 register map.

---

## 3) Register Map Struct

```c
typedef struct
{
  __IO uint32_t CREL;      /*!< offset: 0x000 */
  __IO uint32_t ENDN;      /*!< offset: 0x004 */
  __IO uint32_t RESERVED1; /*!< offset: 0x008 */
  __IO uint32_t DBTP;      /*!< offset: 0x00C */
  ...
  __IO uint32_t CCCR;      /*!< offset: 0x018 */
  ...
} FDCAN_GlobalTypeDef;
```

### Important
- Offsets are **not assigned with `=`** per field.
- Offsets come from struct layout (field order + type sizes + padding).
- `RESERVED` fields are used to keep offsets aligned to the hardware map.

---

## 4) Why `__IO` is used
`__IO` is a CMSIS qualifier (typically `volatile`) for read/write hardware registers.

```c
__IO uint32_t CCCR;
```

Conceptually equivalent to:

```c
volatile uint32_t CCCR;
```

This ensures the compiler performs real memory accesses and does not optimise register reads/writes away.

---

## 5) How register access works

```c
FDCAN1->CCCR |= FDCAN_CCCR_INIT;
```

Interpretation:
1. Start at `FDCAN1_BASE`
2. Add offset of `CCCR` (`0x018`)
3. Perform read-modify-write on that register

So `FDCAN1->CCCR` corresponds to:  
`FDCAN1_BASE + 0x018`

---

## 6) Why this pattern is useful
Instead of raw address code:

```c
*(volatile uint32_t *)(0x4000A018) |= 1;
```

Use CMSIS style:

```c
FDCAN1->CCCR |= FDCAN_CCCR_INIT;
```

Benefits:
- Better readability
- Fewer address/offset mistakes
- Easier maintenance
- Consistent with HAL/CMSIS style

---

## 7) Practical usage rules
- Avoid writing arbitrary values to registers.
- Use bit mask macros (e.g. `FDCAN_CCCR_*`).
- Follow peripheral init/config sequence from reference manual.
- Do not alter reserved bits unless documentation allows it.

---

## 8) Offset verification (optional but useful)

```c
#include <stddef.h>

_Static_assert(offsetof(FDCAN_GlobalTypeDef, CCCR) == 0x18, "CCCR offset mismatch");
_Static_assert(offsetof(FDCAN_GlobalTypeDef, NBTP) == 0x1C, "NBTP offset mismatch");
```

This confirms your struct layout matches expected hardware offsets.

---

## 9) Where definitions usually live
- Device header: `stm32h755xx.h`
- CMSIS core/compiler headers: `core_cm7.h`, `cmsis_compiler.h` (for `__IO`)
- Useful search terms:
  - `FDCAN1_BASE`
  - `#define FDCAN1`
  - `FDCAN_GlobalTypeDef`
  - `#define __IO`

---

## 10) One-line mental model
**Base address + typed struct pointer + ordered fields = named access to real hardware registers.**
```

Estimated effort: 5–10 minutes to commit with a small README link.

**One-line summary:** Here’s a GitHub-ready Markdown note you can paste directly into your repo under `docs/`.  
Next step: if you want, I can also generate a short `README.md` section linking this doc and your learning outcomes.
