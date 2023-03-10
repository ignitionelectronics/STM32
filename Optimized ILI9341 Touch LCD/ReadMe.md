# Optimized ILI9341 LCD with touch

This is a simple library who is optimized for touch and screen. What I have done is that I had made a project that don't use threads and therefore I have optimized this
LCD library so it can write out images very fast using only two SPI. One SPI for the touch and one SPI for the LCD.

This is just examples. Just copy the code and modify it after your own choice. 

Example:

```c
/* Create structure, the J1939 structure is from another project called Open SAE J1939 at my Github */
J1939 j1939 = {0};
STM32_PLC stm32_plc = {0};

/* Init */
STM32_PLC_LCD(&hspi1, &hspi2, LCD_CS_GPIO_Port, LCD_CS_Pin, LCD_DC_GPIO_Port, LCD_DC_Pin, LCD_RST_GPIO_Port, LCD_RST_Pin, TS_CS_GPIO_Port, TS_CS_Pin);

/* Introduction startup */
STM32_PLC_LCD_Show_Intro_Frame();

/* Introduction startup */
STM32_PLC_LCD_Show_Intro_Frame();

/* Check if we touch at the screen */
if(STM32_PLC_LCD_Is_Pressed()){
	/* Blink with all LED lamps before calibration start */
	for(uint8_t i = 0; i < 3; i++){
	 STM32_PLC_LCD_Set_Control_Program(0x1F);
	 HAL_Delay(1000);
	 STM32_PLC_LCD_Set_Control_Program(0x0);
	 HAL_Delay(1000);
	}
	STM32_PLC_LCD_Set_Control_Program(stm32_plc.program_number);
	STM32_PLC_LCD_Calibrate_Touch();
	STM32_PLC_LCD_Get_Touch_Calibration_Parameters(&stm32_plc.Scale_X, &stm32_plc.Scale_Y, &stm32_plc.Bias_X, &stm32_plc.Bias_Y);
	if(!Save_Struct((uint8_t*)&stm32_plc, sizeof(STM32_PLC), STM32_PLC_TEXT_FILE_NAME))
			STM32_PLC_LCD_Show_Information_OK_Dialog("Could not mount SD card");
}

/* Frame start */
uint8_t frame_id = 0;
STM32_PLC_LCD_Show_Main_Frame(&frame_id, false);

while (1)
{
  /* USER CODE END WHILE */

  /* USER CODE BEGIN 3 */

 /* Listen for frame changes */
 STM32_PLC_LCD_Call_Main_Logic(&frame_id, &j1939, &stm32_plc);
 /* Perform control program */
 STM32_PLC_LCD_Execute_Control_Program(&j1939);

}

```

Functionality:

```c
void STM32_PLC_Start_LCD(SPI_HandleTypeDef *lcdSpi, SPI_HandleTypeDef *touchSpi, GPIO_TypeDef *LCD_CS_PORT, uint16_t LCD_CS_PIN, GPIO_TypeDef *LCD_DC_PORT, uint16_t LCD_DC_PIN, GPIO_TypeDef *LCD_RESET_PORT, uint16_t LCD_RESET_PIN, GPIO_TypeDef *TOUCH_CS_PORT, uint16_t TOUCH_CS_PIN);
void STM32_PLC_LCD_Calibrate_Touch();
void STM32_PLC_LCD_Get_Touch_Calibration_Parameters(float *Scale_X, float *Scale_Y, float *Bias_X, float *Bias_Y);
void STM32_PLC_LCD_Set_Touch_Calibration_Parameters(float *Scale_X, float *Scale_Y, float *Bias_X, float *Bias_Y);
void STM32_PLC_LCD_Get_Touch_Data();


/* Frames with no frame id */
void STM32_PLC_LCD_Show_Intro_Frame();
void STM32_PLC_LCD_Show_Main_Frame(uint8_t *frame_id, bool change_only_ABC_buttons);
uint8_t STM32_PLC_LCD_Show_Numpad_Frame(bool decimalbutton_show, bool minusbutton_show, float *number_value, char title[]);
uint8_t STM32_PLC_LCD_Show_Keyboard_Frame(char word[], char title[]);
void STM32_PLC_LCD_Show_Plot_Frame();

/* Frame id 0 */
void STM32_PLC_LCD_Show_Measurement_And_Time_Frame(uint8_t *frame_id);
void STM32_PLC_LCD_Show_Analog_Calibration_Frame(uint8_t *frame_id, STM32_PLC* stm32_plc);
void STM32_PLC_LCD_Show_PWM_Frequency_Settings_Frame(uint8_t *frame_id, STM32_PLC* stm32_plc);

/* Frame id 1 */
void STM32_PLC_LCD_Show_Logging_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_Control_Program_Settings_Frame(uint8_t *frame_id, STM32_PLC* stm32_plc);
void STM32_PLC_LCD_Show_Date_Time_Alarm_Settings_Frame(uint8_t *frame_id);

/* Frame id 2 */
void STM32_PLC_LCD_Show_SAE_J1939_Request_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_Address_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_Commanded_Address_Frame(J1939 *j1939, uint8_t *frame_id);

/* Frame id 3 */
void STM32_PLC_LCD_Show_SAE_J1939_This_ECU_DM1_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_Other_ECU_DM1_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_This_ECU_DM2_Frame(J1939 *j1939, uint8_t *frame_id);

/* Frame id 4 */
void STM32_PLC_LCD_Show_SAE_J1939_Other_ECU_DM2_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_This_ECU_Name_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_Other_ECU_Name_Frame(J1939 *j1939, uint8_t *frame_id);

/* Frame id 5 */
void STM32_PLC_LCD_Show_SAE_J1939_This_ECU_Identifications_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_SAE_J1939_Other_ECU_Identifications_Frame(J1939 *j1939, uint8_t *frame_id);
void STM32_PLC_LCD_Show_Encoder_Revolutions_Settings_Frame(uint8_t *frame_id, STM32_PLC* stm32_plc);

/* Frame id 6 */
void STM32_PLC_LCD_Show_SDADC_Settings_Frame(uint8_t *frame_id, STM32_PLC* stm32_plc);

/* Dialogs */
uint8_t STM32_PLC_LCD_Show_Question_Yes_No_Dialog(char question[]);
uint8_t STM32_PLC_LCD_Show_Information_OK_Dialog(char information[]);

/* Logics */
void STM32_PLC_LCD_Call_Main_Logic(uint8_t *frame_id, J1939 *j1939, STM32_PLC* stm32_plc);
uint8_t STM32_PLC_LCD_Call_Numpad_Logic(bool decimalbutton_show, bool minusbutton_show, float *number_value);
uint8_t STM32_PLC_LCD_Call_Keyboard_Logic(char word[]);
uint8_t STM32_PLC_LCD_Call_Two_Button_Logic(uint16_t b1_x1, uint16_t b1_y1, uint16_t b1_x2, uint16_t b1_y2, uint16_t b2_x1, uint16_t b2_y1, uint16_t b2_x2, uint16_t b2_y2);
uint8_t STM32_PLC_LCD_Call_One_Button_Logic(uint16_t x1, uint16_t y1, uint16_t x2, uint16_t y2);

/* Automatic Control Program */
void STM32_PLC_LCD_Set_Control_Program(uint8_t program_number);
void STM32_PLC_LCD_Set_Program_Parameters(float parameters[]);
uint8_t STM32_PLC_LCD_Get_Control_Program();
void STM32_PLC_LCD_Execute_Control_Program(J1939 *j1939);
```
