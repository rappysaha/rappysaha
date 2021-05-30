---
layout: post
title:  "Xilinx IP - Frame Buffer Read Example"
date:   2021-05-29 
categories: posts-Blog
---
<b>Device name:</b> Zybo. (Zynq-7000)<br>
<b>Vivado Version:</b> 2020.2.<br>
<b>Goal:</b>  To write unique frames in the DDR3 memory capacity and read-out frame by frame as well as display the frames with a variable refresh rate. Real goal is the avoidance of the painful VDMA IP and using a lighter IP.<br>
<b>Github repo:</b> [https://github.com/rappysaha/FrameBufferRead.git](https://github.com/rappysaha/FrameBufferRead.git)<br>

# How to regenerate the project
<b>Step 1:</b> Download the Git repo in your local directory.<br>
<b>Step 2:</b> Vivado 2020.2 should be installed and Zybo boardfiles should be in place. Please follow the [guide from Digilent](https://reference.digilentinc.com/vivado/installing-vivado/start#installing_digilent_board_files) to install the board files perfectly. <br>
<b>Step3:</b> Open the vivado go to the Tcl console. `cd` to the saved location of the github repo<br>

`cd <saved location of git repo>/FrameBufferRead/Prj_FBR/`<br>
press Enter<br>
`source ./FBR.tcl`<br>

vivado will regenerate the project.<br>
<b>Step4:</b> Synthesize, Implement and generate bit stream in the Vivado.<br>
<b>Step5:</b> Export the hardware, .xsa file with bitstream included.<br>
<b>Step6:</b> Setup a Application project with .xsa file that has be exported. Again, you can follow the [guide from the Digilent](https://reference.digilentinc.com/programmable-logic/guides/getting-started-with-ipi#launch_vitis).<br> 
<b>Step7:</b> Copy all the src files from the<br> `<saved location git repo>/FrameBufferRead/Prj_FBR/VitisSrcFIles/`.<br>
<b>Step8:</b> Build the project and connect the Zybo Board. Open the terminal with baudrate 115200. Connect a VGA or HDMI display.<br> 
<b>Step9:</b> Lunch application project in the Zybo by download the .elf file and .bit file in the Zynq Soc. Follow the [Digilent guideline](https://reference.digilentinc.com/programmable-logic/guides/getting-started-with-ipi#launch_a_vitis_baremetal_software_application) here again. <br>

Successfully, you are running the project. <br>

# IP Schematic Design
<img src="{{site.baseurl}}/image/Xilinx IP - Frame Buffer Read Example/IP_Schematic.png" style="width:100%;">

<b>Required IPs:</b><br>

1. Zynq7 Processing System: After block automation, the `S_AXI_HP0` interface was enabled to read out the data from DDR3. The Pl to Ps interrupt was also enabled. The required clocks were also generated from the Zynq also. Three different clocks can be required here.<br>
* M_AXI_GPIO clock: 100 MHz (default). <br>
* S_AXI_HP0 clock: 200MHz. Depends how fast do you want to read out the video. It can be same as M_AXI_GPIO clock, if we do not have any specific read out requirements.<br>
* Clock for VGA and DVI Ips: 74.25 MHz. Depends on output video resolution, in our case it is 720p.<br><br>
<img src="{{site.baseurl}}/image/Xilinx IP - Frame Buffer Read Example/FBR IP Workflow.png" style="float:right;width:30%;">
2. Frame Buffer Read IP: This IP is used to read out frames from the DDR3. <br>
3. AXI4 Stream to Video Out IP: This IP helps to convert AXI4 stream data to the RGB data that will be used by VGA or DVI IP. It can bridge between read out speed and display frequency.<br>
4. The Video Timing Controller: This IP will generate the required timing signals for the desired display resolution.<br>
5. Video Lock Monitor IP: This has input from the AXI4 Stream to Video Out IP. The locked signal from the AXI4 Stream to Video Out IP is monitored by the Video Lock Monitor IP. <br>
6. HLS Reset IP: This will help to reset the Frame Buffer Read IP asynchronously for any time by software. <br>
7. Rgb2vga IP: By this we can connect a VGA display for the output.<br>
8. Rgb2Dvi IP: By this IP we can connect a HDMI display for the output.<br><br><br>

# Vitis Part
After regenerating the project if we move to the Vitis part. In the Vitis we took the bare metal approach. Following description will be helpful to understand the steps of the `main.c` file.<br>

<b>Step 1:</b> Reset the FBR IP,<br>

{% highlight C %}
gpio_hlsIpReset = (uint32_t *)XPAR_HLS_IP_RESET_BASEADDR;
*gpio_hlsIpReset = 1;
{% endhighlight %}

<b>Step 2:</b> Setting up the interrupts and connect it with the ScuGic. Setting up interrupt can be quite confusing some time. If you are using Zynq device, [this presentation](http://indico.ictp.it/event/7987/session/38/contribution/142/material/slides/0.pdf) explains the matter quite clearly and it is really easy to follow. My Approach is more or less similar to the presentation.

{% highlight C %}
Status = SetupInterrupts(ScuGic_ID, FBR_INTR_ID, &intc, &frmbufrd);
if (Status == XST_FAILURE)
{
    xil_printf("ERROR:: Interrupt Setup Failed\r\n");
    xil_printf("ERROR:: Test could not be completed\r\n");
    return (1);
}
{% endhighlight %}

<b>Step 3:</b> Initialize rest of the IPs in your design. We have initialized here video timing controller, FBR and GPIO that is used for check out the locking of the video output.

{% highlight C %}
Status = DriverInit(VTC_ID, FBR_ID, GPIO_ID, &vtc, &frmbufrd, &vmon);
if (Status != XST_SUCCESS)
{
    xil_printf("ERROR:: Driver Initialization Failed\r\n");
    xil_printf("ERROR:: Test could not be completed\r\n");
    return (1);
}
{% endhighlight %}

<b>Step 4:</b> Setting up the callback functions. Most important part of this application. Here, we have connected a function namned `XVFrameBufferCallback()` whenever the there is and interrupt from the FBR IP. Whenever we go to that function the required variables can be accessed via `frmbufrd` structure. 

{% highlight C %}
XVFrmbufRd_SetCallback(&frmbufrd, XVFRMBUFRD_HANDLER_DONE, XVFrameBufferCallback,
                           (void *)&frmbufrd);
{% endhighlight %}

<b>Step 5:</b> Setting up the parameters to configure the Vtc the FBR in the next step. This the place where `#include "xvidc.h"` header file comes handy. Thanks to Xilinx for providing such a wonderful header file to configure the IPs related to the video or image processing. 

{% highlight C %}
VidStream.PixPerClk = frmbufrd.FrmbufRd.Config.PixPerClk;
VidStream.ColorDepth = frmbufrd.FrmbufRd.Config.MaxDataWidth;

// Get video format to test
Cfmt = ColorFormats[0].MemFormat;
VidStream.ColorFormatId = ColorFormats[0].StreamFormat;

// Get mode to test
VidStream.VmId = OutputModes[0];

// Validate testcase format and mode -- Do not need
// Get mode timing parameters
TimingPtr = XVidC_GetTimingInfo(VidStream.VmId);
VidStream.Timing = *TimingPtr;
VidStream.FrameRate = XVidC_GetFrameRate(VidStream.VmId);
{% endhighlight %}

<b>Step 6:</b> At this point, we will write three unique frames to the DDR3 those read out later by FBR IP at different refresh rate and will be displayed at the monitor.
    
{% highlight C %}
// write memory frame by frame
frmPtr = (uint8_t *)XVFRMBUFRD_BUFFER_BASEADDR;
Demo1Frame(0xFE, 0x00, 0x00);
frmPtr = (uint8_t *)(XVFRMBUFRD_BUFFER_BASEADDR + 0x2A3000);
Demo1Frame(0x00, 0xFE, 0x00);
frmPtr = (uint8_t *)(XVFRMBUFRD_BUFFER_BASEADDR + 0x2A3000 + 0x2A3000);
Demo1Frame(0x00, 0x00, 0xFE);
{% endhighlight %}   

<b>Step 7:</b> Configure the vtc according to our expected output video resolution. Then configure the FBR to tell the starting address of the Frame and also notify the FBR how the pixel data is written in the memory. FrameBuffer read IP will be start after that.

{% highlight C %}
// Configure VTC
ConfigVtc(&VidStream);
// Configure Frame Buffer
ConfigFrmbuf(DEMO_STRIDE, Cfmt, &VidStream);
{% endhighlight %}

<b>Step 8:</b> Printing the menu in the terminal and waiting for the user input from the terminal.

{% highlight C %}
PrintMenu();
//go to the while loop
DemoRun();
{% endhighlight %}

Here is the video demo of the project:
<div style="padding:62.5% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/542523167?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="New Recording - 4/28/2021, 4:13:00 PM"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>


