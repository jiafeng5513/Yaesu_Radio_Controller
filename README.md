## 开发说明

- 硬件测试环境:
    - 树莓派
        - 各版本均支持
        - B+和zero采用的单核BCM2835性能稍弱
        - 2B+/3B/3B+/3A+采用的4核CPU性能强, 但功耗更大
        - 请根据自己的预算和电源条件选择
    - GPIO直插LCD屏, 或HDMI外接显示器
        - LCD屏刷新率可能受限于驱动方式和带宽, 请注意识别
    
- 开发测试环境:
    - VMware
    - [DEBIAN STRETCH WITH RASPBERRY PI DESKTOP](https://www.raspberrypi.org/downloads/raspberry-pi-desktop/ "下载地址")

## 调试步骤

### 虚拟机方式

1. 安装VMware并安装Raspbian虚拟机, [参考](https://www.jianshu.com/p/1a65cb0b8f58 "安装参考")

2. 连接到Raspbian, 完成下列设置
    - 使用`raspi-config`
        - 打开SSH功能
        - 如果需使用GPIO屏幕, 请打开SPI及其他所需端口
    - 输入`ip addr`, 查看IP地址
    - `git clone`本仓库或SCP方式将代码下载到虚拟机内

3. 安装依赖Python库, 使用`pip install xxx`(xxx是以下的模块名)
    - pyudev
    - pyserial
    - pytz
    - pyyaml
    - pygame(Raspbian已预装)
    
4. 若物理机为Windows系统, 请下载cp210x驱动, 让宿主机能够识别到电台
    - [下载地址](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers "cp210x驱动下载地址")
    - 使用USB线连接PC和FT-891, 注意打开VMware右下角的USB设备, 使虚拟机能够识别到cp2102

5. 电台设置, CAT_Baudrate=38400

6. 运行代码
    - `python controller.py`

### 树莓派方式

1. 让您的树莓派接入网络
2. 树莓派安装conda
```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh
bash Miniforge3-Linux-aarch64.sh -b -p <where to install miniforge3>
cd <where to install miniforge3>/bin
./conda init
source ~/.bashrc
```
3. 从以上第2步开始


## 代码结构说明

- `rig.py`
    - 设备通信模块, 使用`RIG`类完成设备连接, 以及指令发送
- `controller.py`
    - `Remote_Controller`使用pygame完成前台界面显示(主线程), 
    - `Rig_Polling`线程完成设备轮询, 将设备各参数存入全局变量
- `conf`
    - `custom.yaml`: 设备连接及用户设置
    - `layout.yaml`
        - 界面显示配置, 在主线程中完成资源载入
        - *ELEMENT* 段配置各界面元素使用的图片资源和位置
        - *BUTTON* 段配置各按钮使用的图片资源和显示位置
        - 若需新增分辨率配置, 请将图片资源放在res/{RESOLUTION}路径下, 一个元素的不同状态请垂直排列, *POS*为显示位置, *SIZE*为显示大小, *STAT*为各照片对应的码值
    - `YAESU_CAT3.yaml`
        - 450/891/991/DX系列机型的CAT协议配置
        - 请将功能分解成单一功能进行配置
        - 配置说明参考 `commad_profile_template.yaml`
- `res`
    - 图片资源
    - 建议按分辨率或预设的主题方案进行组织

## FAQ

1. 请问支持哪些机型?

    - 目前支持八重洲450(D)/891/950/991(A)/2000(D)/DX1200/DX3000/DX5000/DX9000, 因为其使用的CAT协议版本相同, 差别较小, 目前选取公共功能部分进行开发.

2. 是否考虑支持817/857/897机型?

    - 个人没有以上机型, 所以暂时没放在开发计划中
    - 这几个机型使用的CAT协议类似
    - 官方目前开放的协议说明不全, 但HRD软件目前拥有较全功能, 故可配合使用逻辑分析仪截取分析
    - 目前的代码框架内, 如果您有8x7系列机型的协议, 可以很方便的实现控制和显示
    - 如果您有完整协议, 也欢迎分享.


3. 频谱功能具体是怎样的?

    - 由于891机型非SDR架构, 且暂无中频引出功能, 故当前的频谱功能和机子内置的SCP功能相同: 扫描一段频率, 根据S表绘制的曲线
    - 扫描时音频有输出, 但由于扫描频率变化很快, 无法分辨语音, 实际实现时会暂时关闭AF增益(即关闭音频输出), 扫描完成后恢复, 效果如原机
    - 在以上实现的基础上, 可以优化信号显示算法, 以及增加便捷功能, 如实现简易的强信号标识、挑选等

4. 为何考虑使用树莓派+Python?

    - 树莓派
        - GPIO丰富, 生态环境较好, 硬件资源丰富
        - 运行完整的linux环境, 软件支持范围广, 可供选择多
        - 性能强大, 最新的3B/3A+, 有4核1.4GHz处理器, 1G/512M内存
    - Python
        - 人生苦短
        - 编程友好, 易入门, 易调试
        - 开发周期短, 在目前代码框架下, 增加一个状态显示, 只需要调整几个配置文件, 放入自己绘制的图标, 外加增加几行代码即可.
    - 缺点
        - 树莓派相比arduino/ESP/STM, 耗电大很多
        - 相比C/汇编, python运行效率低

5. 现阶段, 树莓派能为HAM做什么?

    - 玩玩最火的FT8(难度20%)
        - wsjt有树莓派版本, 额外购买一块声卡, 制作好数据连接线, 即可开始FT8通联
    - 还有其他各种数据通信(难度30%)
        - fldigi是开源软件, 当然可以装在树莓派上运行, 基于它你可以完成大部分数据通信的编解码功能
    - 做个SDR扫描器(难度60%)
        - 开源的RTL-SDR软件也有linux版或源码版, 可以将您的电视棒或其他SDR连接树莓派
    - 为您的各种通联提供各种便利
        - 以上的各种功能, 都可以找到国内外大神提供的代码资源, 整合开发, 完成您自己的软件作品吧
