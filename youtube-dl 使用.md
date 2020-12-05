开源地址：<https://github.com/ytdl-org/youtube-dl>

```bash
参照 https://linuxize.com/post/how-to-install-ffmpeg-on-centos-7/
# centos7 安装 ffmpeg
sudo yum install epel-release
sudo yum localinstall --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm
sudo yum install ffmpeg ffmpeg-devel
ffmpeg -version
```


```bash
# 参照 https://www.sysgeek.cn/youtube-dl-examples/
# 列出单个视频的所有可用的音/视频格式
[root@localhost home]# ./youtube-dl -F https://www.youtube.com/watch?v=xyzabc
[youtube] IwhGr38OBQ4: Downloading webpage
[info] Available formats for xyzabc:
format code  extension  resolution note
249          webm       audio only tiny   52k , opus @ 50k (48000Hz), 4.13MiB
250          webm       audio only tiny   68k , opus @ 70k (48000Hz), 4.82MiB
140          m4a        audio only tiny  130k , m4a_dash container, mp4a.40.2@128k (44100Hz), 10.65MiB
251          webm       audio only tiny  133k , opus @160k (48000Hz), 9.64MiB
394          mp4        256x128    144p   74k , av01.0.00M.08, 30fps, video only, 4.75MiB
278          webm       256x128    144p   78k , webm container, vp9, 30fps, video only, 5.56MiB
160          mp4        256x128    144p   99k , avc1.4d400c, 30fps, video only, 3.46MiB
395          mp4        426x214    240p  159k , av01.0.00M.08, 30fps, video only, 8.76MiB
242          webm       426x214    240p  173k , vp9, 30fps, video only, 9.19MiB
133          mp4        426x214    240p  217k , avc1.4d400d, 30fps, video only, 7.08MiB
396          mp4        640x320    360p  312k , av01.0.01M.08, 30fps, video only, 16.36MiB
243          webm       640x320    360p  367k , vp9, 30fps, video only, 17.33MiB
134          mp4        640x320    360p  476k , avc1.4d401e, 30fps, video only, 12.12MiB
397          mp4        854x428    480p  563k , av01.0.04M.08, 30fps, video only, 28.47MiB
244          webm       854x428    480p  678k , vp9, 30fps, video only, 29.41MiB
135          mp4        854x428    480p  724k , avc1.4d401f, 30fps, video only, 20.15MiB
136          mp4        1280x640   720p 1016k , avc1.4d401f, 30fps, video only, 33.35MiB
398          mp4        1280x640   720p 1166k , av01.0.05M.08, 30fps, video only, 53.26MiB
247          webm       1280x640   720p 1346k , vp9, 30fps, video only, 52.95MiB
399          mp4        1920x960   1080p 2043k , av01.0.08M.08, 30fps, video only, 90.34MiB
248          webm       1920x960   1080p 2361k , vp9, 30fps, video only, 95.23MiB
137          mp4        1920x960   1080p 3852k , avc1.640028, 30fps, video only, 96.15MiB
400          mp4        2560x1280  1440p 6180k , av01.0.12M.08, 30fps, video only, 256.28MiB
271          webm       2560x1280  1440p 7004k , vp9, 30fps, video only, 228.75MiB
401          mp4        3840x1920  2160p 12948k , av01.0.12M.08, 30fps, video only, 553.86MiB
313          webm       3840x1920  2160p 15456k , vp9, 30fps, video only, 703.18MiB
18           mp4        640x320    360p  438k , avc1.42001E, 30fps, mp4a.40.2@ 96k (44100Hz), 36.06MiB
22           mp4        1280x640   720p  534k , avc1.64001F, 30fps, mp4a.40.2@192k (44100Hz) (best)

# 选择上述的最高质量下载，code 码为 401
[root@localhost home]# ./youtube-dl -f 401 https://www.youtube.com/watch?v=xyzabc
[youtube] xyzabc: Downloading webpage
[download] Destination: xxxxxxxxxxxxxxxxxxxxxxx.mp4
[download] 100% of 553.86MiB in 00:28

```
