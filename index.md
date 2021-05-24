# Blog Name:
<!-- <span style="font-family: Times New Roman"> <span style="font-size: 20pt"> SD card initialization and measure the speed of 8, 16, 32, and 64-bit integer data write-read operation in SD-Card by iFZero v1.2 </span> -->
<span style="font-family: Times New Roman"> <span style="font-size: 20pt"> SD-Card Read/Write Speed Check by iFZero AI</span>

## Description:

<span style="font-family: Times New Roman"> <span style="font-size: 15pt">The <b>iFZeroAI</b> board is based on <b>iFZero</b> board <b>(ESP32-PICO-V3 02)</b>. <b>ESP32-PICO-V3 02</b> has featured with 16 GPIOs, 18 RTC_GPIOs, 10 Touch sensors, 8-bit DAC 2 channels, 12-bit ADC 18 channels, 2 channels SPI, 1 channel I2C and so on, for details please check [here](https://www.espressif.com/sites/default/files/documentation/esp32-pico-v3-02_datasheet_en.pdf). It has embedded 8 MB SPI Flash and 2 MB SPI PSRAM, as shown in the following figure.</span>
<table>
  <tr>
    <td align="center" valign="center"><span style="font-size: 15pt;"><ins>ESP32-PICO-V3-02 Block Diagram</ins></span></td>
  </tr>
  <tr>
    <td><img src="esp32-pico-v3-02-blockdiagram.png" style="width:100%"></td>
  </tr>
</table>

<span style="font-family: Times New Roman"> <span style="font-size: 15pt">In this blog, we will show the initialization of <b>SD-Card</b> with writing and reading integer data from 0 to 99 (which are <b>8, 16, 32 and 64-bit</b>) in the <b>SD-Card</b> by using and without using `while()` loop. We do not use `for()` loop because it consumes more time than `while()` loop.</span>

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 1. Here is an overview of the essential peripherals of <b>iFZeroAI</b> board. In the <b><i>Front-side of iFZeroAI</i></b> board, the <b>SD-Card</b>, <b>Camera</b>, <b>MEMS Microphone</b>, <b>TFT Display</b> are shown by yellow, orange, blue and green rectangular box, respectively. In the <b><i>Rare-side of iFZeroAI</i></b> board, the <b> iFZero</b> board, 5-pin USB port, battery port, and switch are shown by dark red, light green, gray, and red rectangular box.</span> </span>

<table>
  <tr>
    <td align="center" valign="center"><span style="font-size: 15pt;"><ins>Front-side of iFZeroAI</ins></span></td>
    <td align="center" valign="center"><span style="font-size: 15pt"><ins>Rare-side of iFZeroAI</ins></span></td>
  </tr>
  <tr>
    <td align="center" valign="center"><img src="iFAI_frontside.png" style="width:50%"></td>
    <td align="center" valign="center"><img src="iFAI_backside.png" style="width:50%"></td>
  </tr>
</table>

<span style="font-family: Verdana"> <span style="font-size: 12pt">Since our topic in this blog is only the measurement of <b>SD-card</b> write-read speed, so we only use the <b>SD-Card</b> slot of the <b>iFZeroAI</b> board. The <b>SD-Card</b> is connected to <b>iFZero</b> board through Serial to Parallel Interface, called <b>SPI</b> protocol. The IO <b>0, 21, 19</b> and <b>22</b> pins of <b>iFZero</b> board are connected to the <b>CS, CLK, MOSI</b> and <b>MISO</b> pins of <b>SD-Card</b>, respectively, as shown in the following figure.</span></span>

<table>
  <tr>
    <td align="center" valign="center"><span style="font-size: 15pt;"><ins>SPI Connection between <b>SD-Card</b> and <b>iFZero Board</b></ins></span></td>
  </tr>
  <tr>
    <td align="center" valign="center"><img src="hardware_connection.png" style="width:50%"></td>
  </tr>
</table>

<span style="font-family: Verdana"> <span style="font-size: 12pt">Before moving to the code, we need to format the SD-Card. First, we insert the SD-card in PC or laptop SD-Card slot. Then follow the following figures to format the SD-Card. Here, we show the SD-Card formatting steps in WIndows 10, version 1909 operating system.</span></span>

<table>
  <tr>
    <td align="center" valign="center"><span style="font-size: 15pt;"><ins>Start SD-Card Formatting</ins></span></td>
    <td align="center" valign="center"><span style="font-size: 15pt"><ins>Finish SD-Card Formatting</ins></span></td>
  </tr>
  <tr>
    <td align="center" valign="center"><img src="md_sd_cardformat_start.png" style="width:100%"></td>
    <td align="center" valign="center"><img src="md_sd_cardformat_finish.png" style="width:100%"></td>
  </tr>
</table>    

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 2. In the Visual Studio Code, we create a project under the <b>Platform IO IDE</b> package. In the <b>main.cpp</b> file, located in <b>src</b> directory, we have to write the codes which will be described in this blog. Now, we write the following necessary header files:</span></span>

```cpp
#include <Arduino.h>
#include <SD.h> // used to initialize the sd card for write and read data in SD card
#include "FS.h" // used for the file-system of SD card
#include "ff.h" // used to read the free and used space of SD card
```

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 3. Declare Pin number of <b>SD-Card</b> to assign in ```SPI()``` function. </span>
    
```cpp
// ************ For SD Card *****************//
#define SD_MISO            22
#define SD_MOSI            19
#define SD_SCLK            21
#define SD_CS              0
// ************ For SD Card *****************//
```

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 4. Declaring the `File` object that will be used to open, write, read and close the <b>(.bin)</b> file in the <b>SD-Card</b>.</span>

```cpp
// ************* Variable for SD Card ************//
File sd_file;
// ************* Variable for SD Card ************//
```

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 5. Variables for 8, 16, 32 and 64-bit data, to write and read in <b>SD-Card</b>. </span>
```cpp
#define total_data_write 100 // total number of data
// ******** for 8-bit write & read **********//
int8_t i_8bit=0;
int8_t read_sddata_8bit[total_data_write];
// ******** for 8-bit write & read **********//
// ******** for 16-bit write & read **********//
int16_t i_16bit=0;
int16_t read_sddata_16bit[total_data_write];
// ******** for 16-bit write & read **********//
// ******** for 32-bit write & read **********//
int32_t i_32bit=0;
int32_t read_sddata_32bit[total_data_write];
// ******** for 32-bit write & read **********//
// ******** for 64-bit write & read **********//
int64_t i_64bit=0;
int64_t read_sddata_64bit[total_data_write];
// ******** for 64-bit write & read **********//
```

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 6. Variables and user-defined function (declaration and definition) to check the size of the <b>SD-Card</b>.</span>

```cpp
uint32_t usedKb, freeKb; // variables
bool SD_getFreeSpace(uint32_t*, uint32_t*); // function declaration

// ********* get the size of SD card *********//
bool SD_getFreeSpace(uint32_t *tot, uint32_t *free)
{
  FATFS *fs;
  DWORD fre_clust, fre_sect, tot_sect;

  /* Get volume information and free clusters of drive 0 */
  if(f_getfree("0:", &fre_clust, &fs) == FR_OK)
  {
    /* Get total sectors and free sectors */
    tot_sect = (fs->n_fatent - 2) * fs->csize;
    fre_sect = fre_clust * fs->csize;
    *tot = tot_sect / 2;
    *free = fre_sect / 2;
    /* Print the free space (assuming 512 bytes/sector) */
    ESP_LOGD(TAG, "%10lu KiB total drive space. %10lu KiB available.", *tot, *free);
    return true;
  }
  return false;
}
// ********* get the size of SD card *********//
```
> Reference: [Get SD card free space not updating?](https://www.esp32.com/viewtopic.php?f=2&t=9914&sid=78355a34d5d8da75d1400404cdb85e48)

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 7. Variables to measure the <b>SD-Card</b> writing and reading speed.</span>

```cpp
unsigned long start_tme, end_tme;
float wrt_spd, red_spd;
```

<span style="font-family: Verdana"> <span style="font-size: 12pt"> 8. Now in `void setup()` function, we initialize the <b>SD-Card</b>, checking <b>SD-Card</b> size, checking <b>SD-Card</b> is ok or not.</span>

```cpp
void setup()
{
  Serial.begin(115200);
  
  // *********** initialize sd card *************
  SPI.begin(SD_SCLK, SD_MISO, SD_MOSI, SD_CS);
  Serial.printf("\n\nInitializing SD card...\n");
  if (!SD.begin(SD_CS))
  {
    Serial.printf("\nSD Init Fail\n");
    delay(2000);
    while(1);
  }
  else
  {
    Serial.printf("\nSD sucessfully initialized\n");
  }
  // *********** initialize sd card *************

  // *********** check sd card size *************
  if(SD_getFreeSpace(&usedKb, &freeKb))
  {
    Serial.printf("\nDrive Space: %.2f GB\nAvailable: %.2f GB\n", float(usedKb)/1024.0/1024.0, float(freeKb)/1024.0/1024.0);
  }
  // *********** check sd card size *************

  // *********** checking sd-card is ok or not ***********
  snprintf(chr_str, sizeof(chr_str), "/data.bin");
  sd_file = SD.open(chr_str, FILE_WRITE);
  Serial.printf("\nFile %s ......", chr_str);
  if(sd_file.print("\n"))
  {
    Serial.printf("...Successfully created.\n");
  }
  else
  {
    Serial.printf("....Failed to create!\n");
  }
  sd_file.close();
  // *********** checking sd-card is ok or not ***********
}
```
    
> <span style="font-family: Verdana"> <span style="font-size: 12pt"> In the serial monitor, the output of the ```void setup()``` code is following: </span>

```cpp
SD successfully initialized

Drive Space: 29.70 GB
Available: 29.70 GB

File /data.bin .........Successfully created.
```

<span style="font-family: Verdana"> <span style="font-size: 12pt">9. Now, we write the code to measure <b>SD-Card</b> write-read speed in the `void loop()` function.</span>

```cpp
void loop()
{
  if(Serial.available())
  {
    input_m = Serial.readString();
    input_m.toCharArray(bufff, 3);
    if(bufff[0] == 'R')
    {
      // ########### data-8bit reading and writing speed #############
      snprintf(chr_str, sizeof(chr_str), "/data_%dbit.bin", sizeof(i_8bit)*8);
      sd_file = SD.open(chr_str, FILE_WRITE);
      // ************ test--write integer in SD card ************
      Serial.printf("\n\nStart writing 8-bit from 0-%d.....", total_data_write-1);
      start_tme = micros();
      i_8bit = 0;
      while(i_8bit < total_data_write)
      {
        sd_file.write((const uint8_t *)&i_8bit, sizeof(i_8bit));
        i_8bit++;
      }
      end_tme = micros();
      wrt_spd = float(sizeof(i_8bit) * 8.0 * i_8bit) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nWriting 0-%d complete--8-bit writing speed: %.2f bit/s", total_data_write-1, wrt_spd);
      // ************ test--write integer in SD card ************
      sd_file.close();

      // ************ test--read data from SD card ************
      Serial.printf("\nStart reading 8-bit from sd card.....");
      sd_file = SD.open(chr_str, FILE_READ);
      int count_read = 0, sdr_ret_val = 1;
      start_tme = micros();
      while(sdr_ret_val > 0)
      {
        sdr_ret_val = sd_file.read((uint8_t *)&read_sddata_8bit[count_read], sizeof(read_sddata_8bit[count_read]));
        count_read++;
      }
      end_tme = micros();
      // Serial.printf("\n time: %.2f", float(end_tme - start_tme));
      red_spd = float(sizeof(read_sddata_8bit[0]) * 8.0) * float(count_read - 1) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nReading finish--8-bit reading speed: %.2f bit/s", red_spd);
      // ************ test--read data from SD card ************
      sd_file.close();
      
      Serial.printf("\nChecking 8-bit read data (0-%d)\n", total_data_write-1);
      for(int j=0; j < count_read-1; j++)
      {
        if(j == count_read-2)
        {
          Serial.printf("%d", read_sddata_8bit[j]);  
        }
        else
        {
          Serial.printf("%d, ", read_sddata_8bit[j]);
        }
      }
      // ########### data-8bit reading and writing speed ############# 


      // ########### data-16bit reading and writing speed #############
      snprintf(chr_str, sizeof(chr_str), "/data_%dbit.bin", sizeof(i_16bit)*8);
      sd_file = SD.open(chr_str, FILE_WRITE);
      // ************ test--write integer in SD card ************
      Serial.printf("\n\n\nStart writing 16-bit from 0-%d.....", total_data_write-1);
      start_tme = micros();
      i_16bit = 0;
      while(i_16bit < total_data_write)
      {
        sd_file.write((const uint8_t *)&i_16bit, sizeof(i_16bit));
        i_16bit++;
      }
      end_tme = micros();
      wrt_spd = float(sizeof(i_16bit) * 8.0 * i_16bit) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nWriting 0-%d complete--16-bit writing speed: %.2f bit/s", total_data_write-1, wrt_spd);
      // ************ test--write integer in SD card ************
      sd_file.close();

      // ************ test--read data from SD card ************
      Serial.printf("\nStart reading 16-bit from sd card.....");
      sd_file = SD.open(chr_str, FILE_READ);
      count_read = 0, sdr_ret_val = 1;
      start_tme = micros();
      while(sdr_ret_val > 0)
      {
        sdr_ret_val = sd_file.read((uint8_t *)&read_sddata_16bit[count_read], sizeof(read_sddata_16bit[count_read]));
        count_read++;
      }
      end_tme = micros();
      red_spd = float(sizeof(read_sddata_16bit[0]) * 8.0) * float(count_read - 1) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nReading finish--16-bit reading speed: %.2f bit/s", red_spd);
      // ************ test--read data from SD card ************
      sd_file.close();

      Serial.printf("\nChecking 16-bit read data (0-%d)\n", total_data_write-1);
      for(int j=0; j < count_read-1; j++)
      {
        if(j == count_read-2)
        {
          Serial.printf("%d", read_sddata_16bit[j]);  
        }
        else
        {
          Serial.printf("%d, ", read_sddata_16bit[j]);
        }
      }
      // ########### data-16bit reading and writing speed #############


      // ########### data-32bit reading and writing speed #############
      snprintf(chr_str, sizeof(chr_str), "/data_%dbit.bin", sizeof(i_32bit)*8);
      sd_file = SD.open(chr_str, FILE_WRITE);
      // ************ test--write integer in SD card ************
      Serial.printf("\n\n\nStart writing 32-bit from 0-%d.....", total_data_write-1);
      start_tme = micros();
      i_32bit = 0;
      while(i_32bit < total_data_write)
      {
        sd_file.write((const uint8_t *)&i_32bit, sizeof(i_32bit));
        i_32bit++;
      }
      end_tme = micros();
      wrt_spd = float(sizeof(i_32bit) * 8.0 * float(i_32bit)) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nWriting 0-%d complete--32-bit writing speed: %.2f bit/s", total_data_write-1, wrt_spd);
      // ************ test--write integer in SD card ************
      sd_file.close();

      // ************ test--read data from SD card ************
      Serial.printf("\nStart reading 32-bit from sd card.....");
      sd_file = SD.open(chr_str, FILE_READ);
      count_read = 0, sdr_ret_val = 1;
      start_tme = micros();
      while(sdr_ret_val > 0)
      {
        sdr_ret_val = sd_file.read((uint8_t *)&read_sddata_32bit[count_read], sizeof(read_sddata_32bit[count_read]));
        count_read++;
      }
      end_tme = micros();
      red_spd = float(sizeof(read_sddata_32bit[0]) * 8.0) * float(count_read - 1) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nReading finish--32-bit reading speed: %.2f bit/s", red_spd);
      // ************ test--read data from SD card ************
      sd_file.close();

      Serial.printf("\nChecking 32-bit read data (0-%d)\n", total_data_write-1);
      for(int j=0; j < count_read-1; j++)
      {
        if(j == count_read-2)
        {
          Serial.printf("%d", read_sddata_32bit[j]);  
        }
        else
        {
          Serial.printf("%d, ", read_sddata_32bit[j]);
        }
      }
      // ########### data-32bit reading and writing speed #############


      // ########### data-64bit reading and writing speed #############
      snprintf(chr_str, sizeof(chr_str), "/data_%dbit.bin", sizeof(i_64bit)*8);
      sd_file = SD.open(chr_str, FILE_WRITE);
      // ************ test--write integer in SD card ************
      Serial.printf("\n\n\nStart writing 64-bit from 0-%d.....", total_data_write-1);
      start_tme = micros();
      i_64bit = 0;
      while(i_64bit < total_data_write)
      {
        sd_file.write((const uint8_t *)&i_64bit, sizeof(i_64bit));
        i_64bit++;
      }
      end_tme = micros();
      wrt_spd = float(sizeof(i_64bit) * 8.0 * float(i_64bit)) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nWriting 0-%d complete--64-bit writing speed: %.2f bit/s", total_data_write-1, wrt_spd);
      // ************ test--write integer in SD card ************
      sd_file.close();

      // ************ test--read data from SD card ************
      Serial.printf("\nStart reading 64-bit from sd card.....");
      sd_file = SD.open(chr_str, FILE_READ);
      count_read = 0, sdr_ret_val = 1;
      start_tme = micros();
      while(sdr_ret_val > 0)
      {
        sdr_ret_val = sd_file.read((uint8_t *)&read_sddata_64bit[count_read], sizeof(read_sddata_64bit[count_read]));
        count_read++;
      }
      end_tme = micros();
      red_spd = float(sizeof(read_sddata_64bit[0]) * 8.0) * float(count_read - 1) * 1000.0 * 1000.0 / float(end_tme - start_tme);
      Serial.printf("\nReading finish--64-bit reading speed: %.2f bit/s", red_spd);
      // ************ test--read data from SD card ************
      sd_file.close();

      Serial.printf("\nChecking 64-bit read data (0-%d)\n", total_data_write-1);
      for(int j=0; j < count_read-1; j++)
      {
        if(j == count_read-2)
        {
          Serial.printf("%lld", read_sddata_64bit[j]);  
        }
        else
        {
          Serial.printf("%lld, ", read_sddata_64bit[j]);
        }
      }
      // ########### data-64bit reading and writing speed #############
      serial_print_once = true;
    }
  }
  else
  {
    if(serial_print_once)
    {
      serial_print_once = false;
      Serial.printf("\n\nPress \"R\" to write, read and check the read and write speed of SD-card\n");
    }
  }
}
```

<span style="font-family: Verdana"> <span style="font-size: 12pt">10. Now, we describe the code of `void loop()` function. The `void loop()` function is an infinite loop that runs forever and if we print anything in serial monitor terminal, we cannot see them consistently in the terminal. As our goal is to measure the write-read speed of <b>SD-Card</b> and print it in such a way so that we can see it in the serial monitor terminal consistently. Therefore, we write the following segment of code so that user can print the speed of write-read operation of <b>SD-Card</b> when user wishes to press "R".</span>

```cpp
if(Serial.available())
{
  input_m = Serial.readString();
  input_m.toCharArray(bufff, 3);
  if(bufff[0] == 'R')
  {
    // here the code of 8, 16, 32 and 64-bit sd-card write-read operation
    ......
  }
}
else
{
  if(serial_print_once)
  {
    serial_print_once = false;
    Serial.printf("\n\nPress \"R\" to write, read and check the read and write speed of SD-card\n");
  }
}
```

<span style="font-family: Verdana"> <span style="font-size: 12pt">The necessary variable for the above segment of code is:</span>

```cpp
String input_m;
char bufff[3];
bool serial_print_once = true;
```
    
> <span style="font-family: Verdana"> <span style="font-size: 12pt"> In the serial monitor, the output looks like following: </span>

```
Press "R" to write, read and check the read and write speed of SD-card
```

<span style="font-family: Verdana"> <span style="font-size: 12pt">11. Now, we describe the code of 8-bit `int` type <b>SD-Card</b> write-read operation and speed measurement of <b>SD-Card</b>. Here, the following code writes and reads from 0 to 99 in <b>SD-Card</b> and measaures the execution time of write-read operation by using `micros()` function. After getting the execution time, we measure the speed in `bit/s`. The code looks like following:</span>
> <b>Noted</b>: Already the following code is shown in ***Section 9*** in `void loop` function, in the `if(bufff[0] == 'R')` block.
```cpp
  // ########### data-8bit reading and writing speed #############
  snprintf(chr_str, sizeof(chr_str), "/data_%dbit.bin", sizeof(i_8bit)*8);
  sd_file = SD.open(chr_str, FILE_WRITE);
  // ************ test--write integer in SD card ************
  Serial.printf("\n\nStart writing 8-bit from 0-%d.....", total_data_write-1);
  start_tme = micros();
  i_8bit = 0;
  while(i_8bit < total_data_write)
  {
    sd_file.write((const uint8_t *)&i_8bit, sizeof(i_8bit));
    i_8bit++;
  }
  end_tme = micros();
  wrt_spd = float(sizeof(i_8bit) * 8.0 * i_8bit) * 1000.0 * 1000.0 / float(end_tme - start_tme);
  Serial.printf("\nWriting 0-%d complete--8-bit writing speed: %.2f bit/s", total_data_write-1, wrt_spd);
  // ************ test--write integer in SD card ************
  sd_file.close();

  // ************ test--read data from SD card ************
  Serial.printf("\nStart reading 8-bit from sd card.....");
  sd_file = SD.open(chr_str, FILE_READ);
  int count_read = 0, sdr_ret_val = 1;
  start_tme = micros();
  while(sdr_ret_val > 0)
  {
    sdr_ret_val = sd_file.read((uint8_t *)&read_sddata_8bit[count_read], sizeof(read_sddata_8bit[count_read]));
    count_read++;
  }
  end_tme = micros();
  // Serial.printf("\n time: %.2f", float(end_tme - start_tme));
  red_spd = float(sizeof(read_sddata_8bit[0]) * 8.0) * float(count_read - 1) * 1000.0 * 1000.0 / float(end_tme - start_tme);
  Serial.printf("\nReading finish--8-bit reading speed: %.2f bit/s", red_spd);
  // ************ test--read data from SD card ************
  sd_file.close();

  Serial.printf("\nChecking 8-bit read data (0-%d)\n", total_data_write-1);
  for(int j=0; j < count_read-1; j++)
  {
    if(j == count_read-2)
    {
      Serial.printf("%d", read_sddata_8bit[j]);  
    }
    else
    {
      Serial.printf("%d, ", read_sddata_8bit[j]);
    }
  }
  // ########### data-8bit reading and writing speed ############# 
```

> The ouput of this code-segment looks like following:

```cpp
Start writing 8-bit from 0-99.....
Writing 0-99 complete--8-bit writing speed: 856531.06 bit/s
Start reading 8-bit from sd card.....
Reading finish--8-bit reading speed: 344679.03 bit/s
Checking 8-bit read data (0-99)
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99
```

* We can see that the 8-bit `int` type data writing speed in <b>SD-Card</b> is about `850 Kb/s`. Here, we write 100 8-bit data 1 by 1 in `while(i_8bit < total_data_write)` loop.
* We read 100 8-bit data from <b>SD-Card</b> 1 by 1 in `while(sdr_ret_val > 0)` loop. In this case, we get the reading speed about `350 Kb/s`.


<span style="font-family: Verdana"> <span style="font-size: 12pt">12. In the same way, we write and read 16-bit `int` type data in <b>SD-Card</b> and get the following Serial output in the terminal.</span>

```cpp
Start writing 16-bit from 0-99.....
Writing 0-99 complete--16-bit writing speed: 661703.88 bit/s
Start reading 16-bit from sd card.....
Reading finish--16-bit reading speed: 604001.50 bit/s
Checking 16-bit read data (0-99)
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99
```

* We can see that the 16-bit `int` type data writing speed in <b>SD-Card</b> is about `662 Kb/s`. Here, we write 100 16-bit data 1 by 1 in `while(i_16bit < total_data_write)` loop.
* We read 100 16-bit <b>SD-Card</b> data 1 by 1 in `while(sdr_ret_val > 0)` loop. In this case, we get the reading speed about `600 Kb/s`.


<span style="font-family: Verdana"> <span style="font-size: 12pt">13. In the same way, we write and read 32-bit and 64-bit `int` type data in <b>SD-Card</b> and get the following Serial output in the terminal.</span>

> Serial output for 32-bit data
```cpp
Start writing 32-bit from 0-99.....
Writing 0-99 complete--32-bit writing speed: 1317957.12 bit/s
Start reading 32-bit from sd card.....
Reading finish--32-bit reading speed: 1356507.00 bit/s
Checking 32-bit read data (0-99)
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99
```

> Serial output for 64-bit data
```cpp
Start writing 64-bit from 0-99.....
Writing 0-99 complete--64-bit writing speed: 1437556.12 bit/s
Start reading 64-bit from sd card.....
Reading finish--64-bit reading speed: 1719043.75 bit/s
Checking 64-bit read data (0-99)
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99
```

* Here, we get writing and reading speed about `1.3 Mb/s` for 32-bit `int` type data.
* For 64-bit `int` type data, we get writing speed: `1.5 Mb/s` and reading speed: `1.7 Mb/s`.


> ### We have found that the <b>SD-Card</b> write-read operation speed is quite high for 64-bit `int` type data than 8, 16, 32-bit `int` type data.

<span style="font-family: Verdana"> <span style="font-size: 12pt">14. To speed-up the <b>SD-Card</b> write-read operation, user must know the size of data, and store them in a predefined array. Then the <b>SD-Card</b> write-read operation can be performed without using `while()` loop and user can also write and read at a time by using `sd_file.write()` and `sd_file.read()` function, respectively. To do this, we first store from 0 to 99 64-bit `int` type data to a `int64_t` datatype array. After taht, pass the array to the `sd_file.write()` function. In the same way, while we are going to read the data from <b>SD-Card</b>, we have to pass an empty `int64_t` type array in `sd_file.read()` function. The code looks like following:</span>

```cpp
  // *********** writing at a time **********
  Serial.printf("\nWrite and read at a time");
  // ########### data-64bit reading and writing speed #############
  snprintf(chr_str, sizeof(chr_str), "/data_%dbit.bin", sizeof(wrt_64bit_arr_insd[0])*8);
  sd_file = SD.open(chr_str, FILE_WRITE);
  // ************ test--write integer in SD card ************
  Serial.printf("\n\n\nStart writing %d-bit from 0-%d.....", sizeof(wrt_64bit_arr_insd[0])*8, total_data_write-1);
  i_16bit = 0;
  while(i_16bit < total_data_write)
  {
    wrt_64bit_arr_insd [i_16bit] = i_16bit;
    i_16bit++;
  }
  start_tme = micros();
  sd_file.write((const uint8_t *)&wrt_64bit_arr_insd, sizeof(wrt_64bit_arr_insd));
  end_tme = micros();
  // Serial.printf("\n time: %.2f", float(end_tme - start_tme));
  wrt_spd = float(sizeof(wrt_64bit_arr_insd) * 8.0) * 1000.0 * 1000.0 / float(end_tme - start_tme);
  Serial.printf("\nWriting 0-%d complete--%d-bit writing speed: %.2f bit/s", total_data_write-1, sizeof(wrt_64bit_arr_insd[0])*8, wrt_spd);
  // ************ test--write integer in SD card ************
  sd_file.close();

  // ************ test--read data from SD card ************
  Serial.printf("\nStart reading %d-bit from sd card.....", sizeof(wrt_64bit_arr_insd[0])*8);
  sd_file = SD.open(chr_str, FILE_READ);
  start_tme = micros();
  sd_file.read((uint8_t *)&read_64bit_arr_frsd, sizeof(read_64bit_arr_frsd));
  end_tme = micros();
  // Serial.printf("\n time: %.2f", float(end_tme - start_tme));
  red_spd = float(sizeof(read_64bit_arr_frsd) * 8.0) * 1000.0 * 1000.0 / float(end_tme - start_tme);
  Serial.printf("\nReading finish--%d-bit reading speed: %.2f bit/s", sizeof(wrt_64bit_arr_insd[0])*8, red_spd);
  // ************ test--read data from SD card ************
  sd_file.close();

  Serial.printf("\nChecking %d-bit read data (0-%d)\n", sizeof(wrt_64bit_arr_insd[0])*8, total_data_write-1);
  for(int j=0; j < total_data_write; j++)
  {
    if(j == total_data_write-1)
    {
      Serial.printf("%lld", read_64bit_arr_frsd[j]);  
    }
    else
    {
      Serial.printf("%lld, ", read_64bit_arr_frsd[j]);
    }
  }
  // ########### data-64bit reading and writing speed #############
  // *********** writing at a time **********
```

<span style="font-family: Verdana"> <span style="font-size: 12pt">The necessary variables for the above code is:</span>

```cpp
int64_t wrt_64bit_arr_insd[total_data_write];
int64_t read_64bit_arr_frsd[total_data_write];
```

> <span style="font-family: Verdana"> <span style="font-size: 12pt">The serial output of the above codes looks like following:</span>

```cpp
Start writing 64-bit from 0-99.....
Writing 0-99 complete--64-bit writing speed: 1831711.50 bit/s
Start reading 64-bit from sd card.....
Reading finish--64-bit reading speed: 2181322.50 bit/s
Checking 64-bit read data (0-99)
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99
```

* Here, we have found that the writing speed is about `1.8 Mb/s` and the reading speed is about `2 Mb/s` which is much higher than the above <b>SD-Card</b> write-read speed, shown from section 11 to 13.

### This kind of code will help to read and write `int` type data in SD-Card in `Mb/s` speed which is fast enough to develop any user-oriented application. 
