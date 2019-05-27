ffmpeg 相关参数定义  
`ffmpeg [global_options] {[input_file_options] -i input_file}...{[output_file_options] output_file}...`
global_options: 全局参数
input_file_options: 输入文件相关参数
output_file_options: 输出文件相关参数

### 视频选项
* -vframes number(ouput): 设置输出文件的帧数，是 -frames:v 的别名
* -vf filtergraph (output): 创建一个 filtergraph 的滤镜链并作用在流上。

### 高级选项
* -discard(input): 允许丢弃特定的流或者分离出的流上的部分帧，但是不是所有的分离器都支持这个特性  
  * none: 不丢帧
  * default：丢弃无效帧
  * noref: 丢弃所有非参考帧
  * bidir: 丢弃所有双向帧
  * nokey: 丢弃所有非关键帧
  * all: 丢弃所有帧

