# ffmpeg 命令备忘 

1. mp4转gif

   ~~~
   ffmpeg -ss 0 -t 5 -i screen.mp4 -s 480*960 -r 15 res.gif
   
   解释：
   表示从0秒开始截取5秒视频转换成gif
   -ss 开始时间偏移，单位秒
   -t 时长，单位秒
   -i 输入的文件 
   -s 大小，size
   -r rate，帧率，单位Hz
   ~~~

   

