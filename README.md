# GPS_Speedometer

Components: STM32WB15CCU Nucleo Microcontroller, NEO-6M GPS module, 1602 LCD display

https://youtube.com/shorts/Z1D6zBesiqY?feature=share

![pic1](https://github.com/JJcompE/GPS_Speedometer/assets/154710856/03cebad3-4439-4620-b8a5-422737d929dd)

# General Explanation of code

While loop running on STM32 board to use intertupt to receive data from GPS module. The UART Rx intertupt callback sets the Flag variable to 1 allowing the function calls to proceed. 

nmea_parse will take data from GPS buffer, parse the NMEA strings to grab the speed, and put the values in the speed variable of the struct myData. 

display_speed will send speeds to LCD display one character at a time.

```
  while (1)
  {
	  HAL_UART_Receive_IT(&huart1, tx_buffer, TX_BUFFER_SIZE);
	  if (Flag ==1) {
		  nmea_parse(&myData, tx_buffer);
		  display_speed();
		  HAL_Delay(100);
	  }
  }
```
Rx interupt callback to set Flag to 1.
```
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    Flag = 1;
}
```

nmea_parse tokenizes the NMEA strings sending them to cooresponding functions to reach each value.

```
void nmea_parse(GPS *gps_data, uint8_t *buffer){
    memset(data, 0, sizeof(data));
    char * token = strtok(buffer, "$");
    int cnt = 0;
    while(token !=NULL){
        data[cnt++] = malloc(strlen(token)+1); //free later!!!!!
        strcpy(data[cnt-1], token);
        token = strtok(NULL, "$");
    }
    for(int i = 0; i<cnt; i++){
       if(strstr(data[i], "\r\n")!=NULL && gps_checksum(data[i])){
           if(strstr(data[i], "GPGLL")!=NULL){
               nmea_GPGLL(gps_data, data[i]);
           }
           else if(strstr(data[i], "GPGSA")!=NULL){
               nmea_GPGSA(gps_data, data[i]);
           }
           else if(strstr(data[i], "GPGGA")!=NULL){
               nmea_GPGGA(gps_data, data[i]);
           }
           else if(strstr(data[i], "GPRMC")!=NULL){
               nmea_GPRMC(gps_data, data[i]);
           }
       }
    }
    for(int i = 0; i<cnt; i++) free(data[i]);
}
```

display_speed first converts the speed from knots to mph then sends each character to the LCD display. 

```
void display_speed(void) {
	char strData[32];
	double speed = myData.speed * 1.15078;

	i2cLcd_ClearDisplay(&h_lcd);
	HAL_Delay(20);
	int i = 0;
	sprintf(strData, "Current Speed:");
	while(strData[i]) {
		i2cLcd_SendChar(&h_lcd, strData[i]);
		i++;
	}
	i = 0;
	i2cLcd_SetCursorPosition(&h_lcd, 0x40);
	sprintf(strData, "%.2lf", speed);
	while(strData[i]) {
		i2cLcd_SendChar(&h_lcd, strData[i]);
		i++;
	}
	Flag =0;
}
```
