# GPS_Speedometer

Components: STM32WB15CCU Nucleo Microcontroller, NEO-6M GPS module, 1602 LCD display

https://youtube.com/shorts/Z1D6zBesiqY?feature=share

![pic1](https://github.com/JJcompE/GPS_Speedometer/assets/154710856/03cebad3-4439-4620-b8a5-422737d929dd)

# General Explanation of code

  while (1)
  {
	  HAL_UART_Receive_IT(&huart1, tx_buffer, TX_BUFFER_SIZE);
	  if (Flag ==1) {
		  nmea_parse(&myData, tx_buffer);
		  //display_lat_long();
		  //HAL_Delay(1000);
		  display_speed();
		  //display_message();
		  HAL_Delay(100);
	  }
	  /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
