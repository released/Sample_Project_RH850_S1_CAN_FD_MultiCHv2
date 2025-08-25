# Sample_Project_RH850_S1_CAN_FD_MultiCHv2
Sample_Project_RH850_S1_CAN_FD_MultiCHv2

update @ 2025/08/25

1. initial __RH850/F1KM-S1 LQFP 64__ , to test below function 

- __UART : RX:P0_3 , TX:P0_2__

- __CAN0 : RX:P0_1 (polling , no RX rule , accept all ID) , TX:P0_0 (polling)__

- __CAN2 : RX:P0_5 (polling , no RX rule , accept all ID) , TX:P0_4 (polling)__

	- use #define ENABLE_CAN0 , ENABLE_CAN2 , to enable multi CH setting( ex : CAN0 , CAN2)

	- use #define CAN_RX_POLLING , #define CAN_RX_INTERRUPT , to change RX receive method

	- check : p->rrt_handle[q].mask.bit.MID=ALL_ID_BIT_IS_NOT_COMPARED;

```c
static void can_rrt_set(CAN_REG_TYP * can,
                          CAN_BUS_HANDLE *p,
                          const CAN_RX_RULE_TABLE_T *rule)
{
    unsigned int q=0;

    for(q=0;q<CAN_RX_RULE_TABLE_AMOUNT;q++)
    {
        //RCFDCnCFDGAFLIDj
        p->rrt_handle[q].id.bit.ID=q;
        p->rrt_handle[q].id.bit.LB=0;
        p->rrt_handle[q].id.bit.IDE=0;
        p->rrt_handle[q].id.bit.RTR=0;
        
        //RCFDCnCFDGAFLMj
        // p->rrt_handle[q].mask.bit.MID=STANDARD_ID_BIT_IS_COMPARED;
        // p->rrt_handle[q].mask.bit.MID=EXTEND_ID_BIT_IS_COMPARED;//The corresponding ID bit is compared
        p->rrt_handle[q].mask.bit.MID=ALL_ID_BIT_IS_NOT_COMPARED;
```

RX use polling or interrupt
```c
// #define CAN_RX_POLLING
#define CAN_RX_INTERRUPT
```

check define : ENABLE_MULTI_CAN_CH
```c
#define ENABLE_MULTI_CAN_CH
```

- __KEYPOINT_1__ : when CAN_RX_FIFO_BUFFER_NUM is NOT CAN_RX_FIFO_BUFFER_NUMBER0 , need to shift the q_number with multiple words 

	- 0x80 byte (128 byte) , which is 32 WORD

```c
volatile CAN_BUS_PARAMETER_T can_bus_parameter_ch4 = 
{
    .CAN_CH                 = CAN_CHANNEL_4,
    .CAN_MODE               = CAN_FD_MIX_MODE,
    .CAN_RX_FIFO_BUFFER_NUM = CAN_RX_FIFO_BUFFER_NUMBER3,
```


can_fd_receive_buffer_decode
```c
    unsigned short q_number = can_bus_parameter_ch4.CAN_RX_FIFO_BUFFER_NUM << 5;
```

can_fd_receive_fifo_buffer_decode
```c   
    unsigned short q_number = rfi_number << 5;
```

- below is register for reference

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/Receive_FIFO_register.jpg)

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/Receive_FIFO_buffer_register.jpg)


- __KEYPOINT_2__ : Sixteen receive rules can be set per page. 

	- each RX channel RULE , align to 16 or each rule must to copy one by one manually

```c
#define CAN_RX_RULE_CURRENT_AMOUNT                  (16)
```

under can_rrt_set_by_channel , initial channel , one by one ,

```c
static void can_rrt_set_by_channel(CAN_REG_TYP * can,
                                    CAN_BUS_HANDLE *p,
                                    CAN_CHANNEL_SEL_e channel,
                                    CAN_RRT_PAGE_SET_e page_sel,
                                    const CAN_RX_RULE_TABLE_T *rule,
                                    unsigned char rule_count)
{
```

```c

void can_rrt_set_all_channel(void)
{
    /* 
        TODO : MUST ASSIGN from PAGE0 and following
        CAN_RX_RULE_TABLE_PAGE0
        CAN_RX_RULE_TABLE_PAGE1
        CAN_RX_RULE_TABLE_PAGE2
        ...
    */
    CAN_RRT_PAGE_SET_e page_cnt = CAN_RX_RULE_TABLE_PAGE0;

    #if defined (ENABLE_CAN0)
    can_rrt_set_by_channel(&RCFDC0,
        &can_bus_handle_ch0,
        can_bus_parameter_ch0.CAN_CH,
        page_cnt,
        can_bus_rx_rule_tbl_ch0,
        CAN_RX_RULE_CURRENT_AMOUNT);
    page_cnt++;
    #endif


```


2. Adjust code flow for easy to add multi channel 

refer to 

```c
void can_init(void)
```

under folder \hal\can_complex.7z , is previous structure for reference

3. Below is PCAN config setting 

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/PCAN_cfg.jpg)

4. Below is power on message

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_MCU_power_on.jpg)

5. when use UART terminal , which send CAN TX message from RH850 , and rececive with PCAN

use digit 1 ~ 4 , to send CAN0 TX data 

use digit 5 ~ 8 , to send CAN2 TX data 

digit 1 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx1.jpg)


digit 2 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx2.jpg)


digit 3 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx3.jpg)


digit 4 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx4.jpg)


digit 5 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx5.jpg)


digit 6 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx6.jpg)


digit 7 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx7.jpg)


digit 8 , 
![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/log_tx8.jpg)


6. Below is different ID test condition , which send by PCAN (refer to PCAN_xfer.xmt) , and rececive with RH850 RX


- ID : 55 , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_055.jpg)


- ID : 66 , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_066.jpg)


- ID : 77 , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_077.jpg)


- ID : 100 , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_100.jpg)


- ID : 103 , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_103.jpg)


- ID : 103 RTR , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_103_RTR.jpg)


- ID : 123 , polling

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_pollig_ID_123.jpg)


- ID : 00 , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_000.jpg)


- ID : 44 , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_044.jpg)


- ID : 7F , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_07F.jpg)


- ID : 88 , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_088.jpg)


- ID : 102_RTR , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_102_RTR.jpg)


- ID : 102_RTR_extend , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_102_RTR_extend.jpg)


- ID : 104 , interrupt

![image](https://github.com/released/Sample_Project_RH850_S1_CAN_FD_MultiCH_CAN0_CAN2/blob/main/rx_interrupt_ID_104.jpg)


7. How to initial interrupt function , 

- Add function declare in r_cg_intvector.c , need to replace when code generage EACH TIMES

SEARCH : extern void eiint23(void);

replace with : extern void can_rx_fifo_interrupt(void);

```c
/* CAN receive FIFO interrupt; */
extern void can_rx_fifo_interrupt(void);
/* CAN0 error interrupt; */
extern void eiint24(void);
```

SEARCH : (void *)eiint23,

replace with : (void *)can_rx_fifo_interrupt,

```c
/* CAN receive FIFO interrupt; */
(void *)can_rx_fifo_interrupt,
/* CAN0 error interrupt; */
(void *)eiint24,
```

- Add interrupt handler declare with irq number 

```c
#pragma interrupt can_rx_fifo_interrupt(enable=false, channel=23, fpu=true, callt=false)
void can_rx_fifo_interrupt(void)
{
    /* Start user code for can_rx_fifo_interrupt. Do not edit comment generated here */
    can_rx_interrupt_cbk();
    /* End user code. Do not edit comment generated here */
}
```

- Add interrupt initial function , 

```c

void R_CANFD_Interrupt_Control_Init(void)
{
	// 23   ICRCANGRECC0  	FFFE EA2E H   INTRCANGRECC0  	CAN receive FIFO interrupt

    /*INTRCANGRECC0 : CAN CHANNEL RX FIFO interrupt*/

    INTC1.ICRCANGRECC0.BIT.CTRCANGRECC0 = 1;
    INTC1.ICRCANGRECC0.BIT.RFRCANGRECC0 = _INT_REQUEST_NOT_OCCUR;        
    INTC1.ICRCANGRECC0.BIT.MKRCANGRECC0 = _INT_PROCESSING_ENABLED;
    INTC1.ICRCANGRECC0.BIT.TBRCANGRECC0 = _INT_TABLE_VECTOR; //select table interrupt

    INTC1.ICRCANGRECC0.BIT.P3RCANGRECC0 = 1;
    INTC1.ICRCANGRECC0.BIT.P2RCANGRECC0 = 1;
    INTC1.ICRCANGRECC0.BIT.P1RCANGRECC0 = 1;
    INTC1.ICRCANGRECC0.BIT.P0RCANGRECC0 = 1;
}
