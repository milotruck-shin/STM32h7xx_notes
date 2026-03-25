/* Exported macro ------------------------------------------------------------*/
#ifdef  USE_FULL_ASSERT
/**
  * @brief  The assert_param macro is used for function's parameters check.
  * @param  expr: If expr is false, it calls assert_failed function
  *         which reports the name of the source file and the source
  *         line number of the call that failed.
  *         If expr is true, it returns no value.
  * @retval None
  */
  #define assert_param(expr) ((expr) ? (void)0U : assert_failed((uint8_t *)__FILE__, __LINE__))
/* Exported functions ------------------------------------------------------- */
  void assert_failed(uint8_t *file, uint32_t line);
#else
  #define assert_param(expr) ((void)0U)
#endif /* USE_FULL_ASSERT */


////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/*how to check the parameters of a enum*/
#include <stdint.h>

// 1) Enum
typedef enum
{
    MOTOR_MODE_STOP = 0,
    MOTOR_MODE_RUN  = 1,
    MOTOR_MODE_CAL  = 2
} MotorMode_t;

// 2) Validity macro  - here, \ is a newline macro format to say the macro is continued on the next line. 
//this macro returns true if mode is equal to one of the allowed enum constants.

#define IS_MOTOR_MODE(MODE) (((MODE) == MOTOR_MODE_STOP) || \
                             ((MODE) == MOTOR_MODE_RUN)  || \
                             ((MODE) == MOTOR_MODE_CAL))

// 3) assert_param macro (typically from stm32xxxx_hal_conf.h)
#ifdef USE_FULL_ASSERT
void assert_failed(uint8_t *file, uint32_t line);
#define assert_param(expr) ((expr) ? (void)0U : assert_failed((uint8_t *)__FILE__, __LINE__))
#else
#define assert_param(expr) ((void)0U)
#endif

// 4) Use in API
void Motor_SetMode(MotorMode_t mode)
{
    assert_param(IS_MOTOR_MODE(mode));
    // continue safely...
}
