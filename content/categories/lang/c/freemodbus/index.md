+++
title= "Freemodbus"
description= "freemodbus的使用"
date= 2022-04-19T09:01:21+08:00
author= "chao"
draft= true
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    "project","c"
]

+++

# freemodus

## 源码

[cwalter-at/freemodbus](https://github.com/cwalter-at/freemodbus)

[armink/FreeModbus_Slave-Master-RTT-STM32: Add master mode to FreeModbus. | 在 FreeModbus 中添加主机模式 (github.com)](https://github.com/armink/FreeModbus_Slave-Master-RTT-STM32)

## modbus



|      |                  |
| ---- | :--------------: |
| 0x01 |   读线圈寄存器   |
| 0x02 | 读离散输入寄存器 |
| 0x03 |   读保持寄存器   |
| 0x04 |   读输入寄存器   |
| 0x05 | 写单个线圈寄存器 |
| 0x06 | 写单个保持寄存器 |
| 0x0f | 写多个线圈寄存器 |
| 0x10 | 写多个保持寄存器 |

###### 线圈寄存器（0x01、0x05、0x0f）

> 寄存器的单位是1bit,类比为开关量，每一个bit都对应一个信号的开关状态，1byte有8bit

###### 离散输入寄存器（0x02）

> 离散输入寄存器就相当于线圈寄存器的只读模式

###### 保持寄存器（0x03、0x06、0x10）

> 寄存器的单位是2byte,即16bit

###### 输入寄存器（0x04）

> 和保持寄存器类似，但是也是只支持读而不能写

###### modbus地址规范

> 00001至09999是离散输出(线圈)-----Coil status
> 		10001至19999是离散输入(触点)-----Input status
> 		30001至39999是输入寄存器(通常是模拟量输入)------Input register
> 		40001至49999是保持寄存器 -------Holding register



<br>

&emsp;&emsp;0x代表线圈（DO）类地址，1x代表触点（DI）类地址、 3x代表输入寄存器（AI）类地址、4x代表输出寄存器（AO）类地址。在实际编程中，由于前缀的区分作用，所以只需说明后4位数，而且需转换为4位十六进制地址

## Linux的使用

&emsp;&emsp;官方规定的流程：

~~~
 * // Initialize protocol stack in RTU mode for a slave with address 10 = 0x0A
 * eMBInit( MB_RTU, 0x0A, 38400, MB_PAR_EVEN );
 * // Enable the Modbus Protocol Stack.
 * eMBEnable(  );
 * for( ;; )
 * {
 *     // Call the main polling loop of the Modbus protocol stack.
 *     eMBPoll(  );
 *     ...
 * }
~~~

### RTU

~~~
#include <iostream>

#include "mb.h"
#include "mbport.h"
#define REG_INPUT_START 1000    /* 输入寄存器开始地址 */
#define REG_INPUT_NREGS 4     /* usRegInputBuf数组大小 */
#define REG_HOLDING_START 2000  /* 保持寄存器开始地址 */
#define REG_HOLDING_NREGS 130  /* usRegHoldingBuf数组大小 */

int main()
{
    /*RTUm模式 从机地址 uart 波特率 奇偶无校验 */
    eMBInit( MB_RTU, 0x05,0,115200, MB_PAR_NONE );

    if(eMBEnable( )== MB_ENOERR)
    {  
        for( ;; )
        {
            // Call the main polling loop of the Modbus protocol stack.
            if( eMBPoll(  ) != MB_ENOERR )
            break;
        }
        std::cerr<<"err: eMBPoll failed!";
    }
    else
    {
        std::cerr<<"err: eMBEnable failed!";
    }
    return 0;
}

/* ----------------------- Static variables ---------------------------------*/
static USHORT   usRegInputStart = REG_INPUT_START;
static USHORT   usRegInputBuf[REG_INPUT_NREGS]={0x1,0x2,0x3,0x4}; //输入寄存器数组
static USHORT   usRegHoldingStart = REG_HOLDING_START;
static USHORT   usRegHoldingBuf[REG_HOLDING_NREGS];//保持寄存器数组


/* 输入寄存器 */
eMBErrorCode  eMBRegInputCB( UCHAR * pucRegBuffer, USHORT usAddress,USHORT usNRegs )
{
    
    eMBErrorCode    eStatus = MB_ENOERR;
    int             iRegIndex;
    std::cout<<"enter\n";

    /**
     * 满足条件：
     *  1. 主机请求地址大于REG_INPUT_START
     *  2. 主机请求数据量不超过 usRegInputBuf数组大小
     */
    if( ( usAddress >= REG_INPUT_START )
    && ( usAddress + usNRegs <= REG_INPUT_START + REG_INPUT_NREGS ) )
    {
        iRegIndex = ( int )( usAddress - usRegInputStart );
        while( usNRegs > 0 )
        {
            *pucRegBuffer++ = ( unsigned char )( usRegInputBuf[iRegIndex] >> 8 );
            *pucRegBuffer++ = ( unsigned char )( usRegInputBuf[iRegIndex] & 0xFF );
            iRegIndex++;
            usNRegs--;
        }
    }
    else
    {
        eStatus = MB_ENOREG;
    }

    return eStatus;
}

/* 保持寄存器 */
eMBErrorCode   eMBRegHoldingCB( UCHAR * pucRegBuffer, USHORT usAddress,USHORT usNRegs, eMBRegisterMode eMode )
{

}
/* 线圈寄存器 */
eMBErrorCode   eMBRegCoilsCB( UCHAR * pucRegBuffer, USHORT usAddress,USHORT usNCoils, eMBRegisterMode eMode )
{

}
/* 离散输入寄存器 */
eMBErrorCode    eMBRegDiscreteCB( UCHAR * pucRegBuffer, USHORT usAddress,USHORT usNDiscrete )
{

}

~~~





