title: RTL-sdr
date: 1912-12
categories: [sdr]
tags: [rtl-sdr]
photos:
---
文章摘要
<!--more-->

fcd_nfm_rx --freq 95800
lspci

 1135  rtl_power
 1136  rtlsdr-scanner 
 1137  rtl_fm
 1138  rtl_fm -h
 1139  trl_fm -f 95.8 -s 22050 | multimon -t raw /dev/stdin
 1140  rtl_fm -f 95.8 -s 22050 | multimon-ng -t raw /dev/stdin
 1141  rtl_fm -f 95.8M -s 22050 | multimon-ng -t raw /dev/stdin
 1142  rtl_fm -f 95.77M -s 24000 | multimon-ng -t raw /dev/stdin










./bettercap  -I wlan0 -T 192.168.1.104 -X




