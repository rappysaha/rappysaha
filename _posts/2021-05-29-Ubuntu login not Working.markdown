---
layout: post
title:  "Ubuntu login not Working in VM."
date:   2021-05-29 
categories: jekyll update
---
I was working with Virtualbox 6.1 with ubuntu distribution 18.04. The display resolution was too small and I wanted to change it by following this [post](https://www.nakivo.com/blog/make-virtualbox-full-screen/#:~:text=In%20order%20to%20fix%20this,CD%20drive%20of%20the%20VM.&text=Now%20you%20can%20maximize%20the,of%20the%20Windows%2010%20guest.). I did not follow accordingly, and I just tried to install Guest edition from the following option.
<tr>
  <td><img src="screenshot1.png" style="width:100%"></td>
</tr>
<br>
<img src="{{site.baseurl}}/_posts/screenshot1.png">

It messed up my full distro. After installation and rebooting the ubuntu, even if I was giving the right password, I cannot pass the login screen. <br><br>
<b>Solution</b>
<br>Press `ctrl+alt+F3` for the tty login page. (This can be ctrl+alt+(press any button in between F1-F7, see which one works for you))<br>
Give username. Press Enter.<br>
Give the Password. Press Enter.<br>
If you can login, there is still hope for you. <br>

Run `df -h` <br>
{% raw %}
<a href="screenshot2.png" title="View larger picture"><img src="screenshot2.png" alt="screenshot2.png"style="width:100%;"/></a>
{% endraw %}
We can see the `/dev/sda1` that is actual drive that we have created during the installation. This screenshot was taken when the problem was solved. When I was facing the problem there was extra `/home` folder in the column of Mounted on (that is not showing when the problem was solved).<br>
Tried to cd `/home` but no permission.<br>

`sudo ls -al /home`<br>
<br>
In the `/home` folder I could not see my user folder rpprnx (username). At this point, we can understand our distro is not booting from our desired sata drive (in my case `/dev/sda1`). To see the details of `dev/sda1`,<br>

`sudo mount /dev/sda1 /home`<br>
`sudo ls -al /home`<br>

Oh! NOW I can see my rpprnx folder in the location of `/home/home/rpprnx`. To solve the problem, I ran following commands from this [post](https://askubuntu.com/questions/882385/dev-sda1-clean-this-message-appears-after-i-startup-my-laptop-then-it-w). 

`sudo apt update`<br>
`sudo apt clean`<br>
`sudo apt autoremove`<br>
<br>Run `df -h` <br>
We will see the above screen shot with out any extra `/home` folder.