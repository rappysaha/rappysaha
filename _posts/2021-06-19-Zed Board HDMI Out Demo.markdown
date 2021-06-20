---
layout: post
title:  "Zed Board HDMI Out Demo"
date:   2021-06-19
categories: posts-Blog
---
<b>Device name:</b> ZED Board. (Zynq-7000)<br>
<b>Vivado Version:</b> 2020.2.<br>
<b>Goal:</b>  Test the Zed board HDMI port. <br>
<b>Github repo:</b> [https://github.com/rappysaha/Zed_HDMI_OUT.git](https://github.com/rappysaha/Zed_HDMI_OUT.git)<br>

# How to regenerate the project
<b>Step 1:</b> Download the Git repo in your local directory.<br>
<b>Step 2:</b> Vivado 2020.2 should be installed and Zed board boardfiles should be in place. Please follow the [guide from Digilent](https://reference.digilentinc.com/vivado/installing-vivado/start#installing_digilent_board_files) to install the board files perfectly. For this project the license for video test pattern generator IP will be required. Evaluation license will work also. <br>
<b>Step3:</b> Open the vivado go to the Tcl console. `cd` to the saved location of the github repo<br>

`cd <saved location of git repo>/Zed_HDMI_OUT/`<br>
press Enter<br>
`source ./create_proj.tcl`<br>

vivado will regenerate the project.<br>
<b>Step4:</b> Synthesize, Implement and generate bit stream in the Vivado.<br>
<b>Step5:</b> Export the hardware, .xsa file with bitstream included.<br>
<b>Step6:</b> Setup a Application project with .xsa file that has be exported. Again, you can follow the [guide from the Digilent](https://reference.digilentinc.com/programmable-logic/guides/getting-started-with-ipi#launch_vitis).<br> 
<b>Step7:</b> Copy all the src files from the<br> `<saved location git repo>/Zed_HDMI_OUT/vitisSrcFile/`.<br>
<b>Step8:</b> Build the project and connect the Zed Board. Open the terminal with baudrate 115200. Connect a HDMI display.<br> 
<b>Step9:</b> Lunch application project in the Zed by download the .elf file and .bit file in the Zynq Soc. Follow the [Digilent guideline](https://reference.digilentinc.com/programmable-logic/guides/getting-started-with-ipi#launch_a_vitis_baremetal_software_application) here again. <br>

Successfully, you are running the project. <br>

NB: During the synthesis, if you have any warning similar to the following warning, reducing the path length of the project will help.

`WARNING: [IP_Flow 19-4318] IP-XACT file does not exist: d:/Test/Zed_HDMI_OUT/proj_1/proj_1.srcs/sources_1/bd/ZED_Hdmi_Out/ip/ZED_Hdmi_Out_v_tpg_0_0/ZED_Hdmi_Out_v_tpg_0_0.xml. It will be created.`

# IP Schematic Design
<!-- <img src="{{site.baseurl}}/image/Zed Board HDMI Out Demo/IP_Schematic.png" style="width:100%;"> -->
<a href="{{site.baseurl}}/image/Zed Board HDMI Out Demo/IP_Schematic.png" title="View larger picture"><img src="{{site.baseurl}}/image/Zed Board HDMI Out Demo/IP_Schematic.png" alt="IP Schematic"
style="width:100%;"/></a>

<b>Required IPs:</b><br>

1. Zynq7 Processing System: It will help to connect Axi based pripherall to control the parameters from the OS side. In our case, AXI based IIC controller and video Test pattern generator parameters will be controlled by the Zynq processing system in the OS side. 
2. AXI IIC IP: This I2C IP is the main requirement for the Zed board HDMI configuration. The I2C IP is required to configure the ADV7511 chip. We need to supply the 16-bit video data to the ADV7511 since it will accept only 16-bit from the FPGA. In the XDC file of the Zed Board, we can see the 16-bit data connection for the HDMI part.
3. Video Test Pattern Generator IP: The test pattern generator will generate 720p test pattern. The clock of the video timing controller will depend on the selected resolution. 
4. Video Timing Controller: The timing controller for this example is in the native mode to keep the example as simple as possible. 
5. AXI4 Stream to Video Out IP: Video Out IP will be configured in the independent mode. The video out clock will be same as the video timing controller clock. The output of the Video out IP will be connected to the ADV7511 input. In this example, we are not considering audio output via HDMI. 

# Vitis Part
After regenerating the project, if we move to the Vitis part. In the Vitis, we took the bare metal approach. Following description will be helpful to understand the steps of the `main.c` file.<br>

<b>Step 1:</b> Initialize the AXI IIC interface<br>

{% highlight C %}
Status = zed_iic_axi_init( &iic, "ZED HDMI I2C Controller",IIC_BASEADDR);

if (Status != 1) {
    xil_printf( "ERROR : Failed to initialize IIC driver\n\r" );
    return -1;
}
xil_printf("IIC initialization Successful\n");
{% endhighlight %}

<b>Step 2:</b> Hot plug detection system. It is not an interrupt based system, usually hot plug detection system should be an interrupt based system. Therefore, it will detect the HPD signal for once.<br>

{% highlight C %}
while(!mon_conn)
{
    mon_conn = check_hdmi_hpd_status(&iic, 0x39);
    if (mon_conn)
    {
        xil_printf("HDMI Connected\n");
    }
    else
    {
        xil_printf("HDMI NOT - Connected\n");
    }
    sleep(2);
}
{% endhighlight %}

<b>Step 3:</b> Configuring particular registers of the ADV7511 via I2C. These are the predefined registers, defined by the Avnet.<br>

{% highlight C %}
HDMI_config(&iic);
xil_printf("HDMI Setup Complete!\r\n");
{% endhighlight %}

<b>Step 4:</b> Initializing test pattern generator (TPG), configuring TPG and Start TPG<br>

{% highlight C %}
Status = XV_tpg_Initialize(&tpg, TPG_ID);
if(Status!= XST_SUCCESS)
{
    xil_printf("TPG configuration failed\r\n");
    return(XST_FAILURE);
}
// Set Resolution to 1280x720
XV_tpg_Set_height(&tpg, 720);
XV_tpg_Set_width(&tpg, 1280);

// Set Color Space to YUV422
XV_tpg_Set_colorFormat(&tpg, 0x2);

XV_tpg_Set_bckgndId(&tpg, XTPG_BKGND_COLOR_BARS);
// Set	 Overlay to moving box
// Set the size of the box
XV_tpg_Set_boxSize(&tpg, 50);
// Set the speed of the box
XV_tpg_Set_motionSpeed(&tpg, 5);
// Enable the moving box
XV_tpg_Set_ovrlayId(&tpg, 1);
//Start the TPG
XV_tpg_EnableAutoRestart(&tpg);
XV_tpg_Start(&tpg);
{% endhighlight %}

The LED 0 of the Zed board will be turned on when video output will be locked. At the end, no specific HDMI IP is required for the Zed Board HDMI since ADV7511 is doing most of the work. On the other hand, board like Zybo do not have this ADV7511 chip, therefore, RGB2DVI IP from Digilent was quite helpful. 

# Demo
The Zed Board connection:
<a href="{{site.baseurl}}/image/Zed Board HDMI Out Demo/Zed Board Connection.jpg" title="View larger picture"><img src="{{site.baseurl}}/image/Zed Board HDMI Out Demo/Zed Board Connection.jpg" alt="IP Schematic"
style="width:100%;"/></a>


Here is the video demo of the project:
<div style="padding:62.5% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/564511236?badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="ZedBoard HDMI Test - 6/18/2021, 4:06:36 PM"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<br>

# References:
The project was created by the help of the following links. 

[Xilinx Video Series 19](https://forums.xilinx.com/t5/Video-and-Audio/Video-Series-19-Using-the-On-Board-HDMI-on-ZC702-Vivado-design/td-p/914989)

[Xilinx Video Series 20](https://forums.xilinx.com/t5/Video-and-Audio/Video-Series-20-Starting-with-SDK-and-configuring-the-ADV7511/m-p/917308#M23024%2Fjump-to%2Ffirst-unread-message)

[Xilinx Video Series 21](https://forums.xilinx.com/t5/Video-and-Audio/Video-Series-21-TPG-Application-on-ZC702/td-p/922324)

The above mentioned series targeted ZC702 board while this project is all about Zed board. Lots of similarities but simple difference is there and that is also discussed in the [Xilinx forum](https://forums.xilinx.com/t5/Video-and-Audio/Zedboard-HDMI-Display-Failed-to-Access-ADV7511-via-I2C/td-p/1054540) by the author of the above series. Depending on this discussion this project was build. There is slight modification in HDMI configuration.