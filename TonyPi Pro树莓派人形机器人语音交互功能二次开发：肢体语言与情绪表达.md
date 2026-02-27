# TonyPi Pro树莓派人形机器人语音交互功能二次开发：肢体语言与情绪表达
## 1. 修改概述
为满足人形机器人能够给人增加情绪价值，在原有语音交互基础上增加两个新功能：
- **点阵显示**：支持机器人初始化显示时间及机器人在唤醒、思考、回答时的情绪表达
- **动作组**：支持机器人在初始化，唤醒、思考、回答时的肢体语言
- 该项目位于 /home/pi/large_models/interaciton_demo.py
## 2. 文件修改总览
| 修改类型 | 位置 | 说明 |
|---------|------|------|
| 新增功能:点阵显示 | 第39-106行 | `run()` - 点阵显示 |
| 新增功能:执行动作组 | 第110-123行 | `action()` - 动作组 |
| 新增导入 | 第12-15行 | 引入点阵模块、多线程、动作组，时间日期模块 |
| 新增变量 | 第127-128行 | 线程锁、重复开启判断 |
| 修改原功能 | 第126-188行 | 多线程同时控制语音、点阵、动作组 |
## 3. 新增功能详解

### 3.1 功能A：点阵显示
#### 设计思路
当人形机器人初始化或者静默时，点阵屏显示当前时间（北京时间）；当机器人被唤醒的时候，点阵屏显示被唤醒的动画；当机器人收到语音时，点阵屏显示思考的动画，当机器人回复的时候，点阵屏显示眨眼动画
#### 代码实现
**python代码实现：**
```python
def run(runner, m):
    dc = {
        0: (0x3E, 0x22, 0x3E),
        1: (0x00, 0x3E, 0x00),
        2: (0x3A, 0x2A, 0x2E),
        3: (0x2A, 0x2A, 0x3E),
        4: (0x0E, 0x08, 0x3E),
        5: (0x2E, 0x2A, 0x3A),
        6: (0x3E, 0x2A, 0x3A),
        7: (0x02, 0x02, 0x3E),
        8: (0x3E, 0x2A, 0x3E),
        9: (0x2E, 0x2A, 0x3E),
    }  # 设置字典（键值对），不同数字选择不同的字模数组，查询时返回数据结构是元组
    dms = dot_matrix_sensor.TM1640(dio=7, clk=8)  # 初始化点阵
    while runner:
        now = datetime.now()
        current_time = now.strftime("%H:%M")  # time
        time_digits = [int(char) for char in current_time if char.isdigit()]
        signal_time = dc[time_digits[0]] + (0x00,) + dc[time_digits[1]] + (0x00,) + (0x00,) + dc[time_digits[2]] + (
            0x00,) + dc[time_digits[3]]
        # 获取时间并拼接成点阵屏上的字模数组（数据结构：元组）
        try:
            # 点阵状态0：显示时间实时更新
            if m[0] == 0:
                dms.display_buf = signal_time
                dms.update_display()
            # 点阵状态1：唤醒
            if m[0] == 1:
                dms.display_buf = (0x3C, 0x7E, 0xEF, 0xF7, 0xFF, 0x7E, 0x3C, 0x00, 0x00, 0x3C, 0x7E, 0xF7, 0xEF, 0xFF,
                                   0x7E, 0x3C)
                dms.update_display()
                time.sleep(0.2)
                dms.display_buf = (0x3C, 0x7E, 0xF7, 0xEF, 0xFF, 0x7E, 0x3C, 0x00, 0x00, 0x3C, 0x7E, 0xEF, 0xF7, 0xFF,
                                   0x7E, 0x3C)
                dms.update_display()
                time.sleep(0.2)
                dms.display_buf = (0x3C, 0x7E, 0xFF, 0xFF, 0xEF, 0x76, 0x3C, 0x00, 0x00, 0x3C, 0x7E, 0xFF, 0xEF, 0xF7,
                                   0x7E, 0x3C)
                dms.update_display()
                time.sleep(0.2)
            # 点阵状态2：思考
            if m[0] == 2:
                dms.display_buf = (0x7E, 0x81, 0x85, 0x91, 0x81, 0x7E, 0x00, 0x00, 0x00, 0x00, 0x7E, 0x81, 0x85, 0x91,
                                   0x81, 0x7E)
                dms.update_display()
                time.sleep(0.1)
                dms.display_buf = (0x7E, 0x81, 0x95, 0x81, 0x81, 0x7E, 0x00, 0x00, 0x00, 0x00, 0x7E, 0x81, 0x95, 0x81,
                                   0x81, 0x7E)
                dms.update_display()
                time.sleep(0.1)
                dms.display_buf = (0x7E, 0x81, 0x91, 0x85, 0x81, 0x7E, 0x00, 0x00, 0x00, 0x00, 0x7E, 0x81, 0x91, 0x85,
                                   0x81, 0x7E)
                dms.update_display()
                time.sleep(0.1)
            # 点阵状态3：眨眼回答
            if m[0] == 3:
                dms.display_buf = (0x18, 0x18, 0x18, 0x18, 0x18, 0x18, 0x18, 0x00, 0x00, 0x18, 0x18, 0x18, 0x18, 0x18,
                                   0x18, 0x18)
                dms.update_display()
                time.sleep(0.2)
                dms.display_buf = (0x3C, 0x7E, 0xFF, 0xFF, 0xFF, 0x7E, 0x3C, 0x00, 0x00, 0x3C, 0x7E, 0xFF, 0xFF, 0xFF,
                                   0x7E, 0x3C)
                dms.update_display()
                time.sleep(0.2)
        except:
            dms.display_buf = [0] * 16
            dms.update_display()
            break
```
#### 注意事项
**点阵屏如果在多个线程被使用会出现阻塞，所以，不要再添加新的点阵线程！！！**
### 3.2 功能B：执行动作组
#### 设计思路
当人形机器人初始化或者静默时，机器人手臂抬起，让点阵屏对准前方；当机器人被唤醒的时候，机器人举手表示“我在”；当机器人收到语音时，挠头思考，当机器人回复的时候，挥手解答
#### 代码实现
**python代码实现：**
```python
def action(actioner, m):
    while actioner:

        try:
            if m[0] == 0:
                AGC.runActionGroup('interaciton1')  # 动作状态0:抬手
            if m[0] == 1:
                AGC.runActionGroup('interaciton2')  # 动作状态1：举手
            if m[0] == 2:
                AGC.runActionGroup('interaciton3')  # 动作状态2：挠头思考
            if m[0] == 3:
                AGC.runActionGroup('interaciton4')  # 动作状态3：挥手回答
        except:
            break
```
#### 注意事项
**请勿在开机时强行掰舵机，以免造成损害**
## 4. 修改原功能（其它代码保持不变）
**python代码实现：**
```python
def main():
    i = 0  # 变量i决定是开机自启状态(站立)，还是已经提问过了(抬手臂)，并且由于线程阻塞的原因，点阵模块和动作组都只能开一个线程，需要一个变量来判断是否会重复开启
    m = [0]  # 变量m决定线程状态，该做哪个动作或者点阵屏显示什么图案
    kws.start()
    while True:
        m[0] = 0  # 动作状态0:抬手，点阵状态0：显示时间实时更新
        runner = True  # 点阵屏和动作组共用一个线程锁（开）

        if i == 0:
            AGC.runActionGroup('interaciton0')  # 由于抬手臂不能一气呵成，所以需要一个预备动作
            th = threading.Thread(target=run, args=(runner, m))  # 设置点阵线程
            th.daemon = True  # 设置为守护线程，主线程结束后子线程自动结束
            th.start()  # 开启点阵线程
            th2 = threading.Thread(target=action, args=(runner, m))  # 设置动作线程
            th2.daemon = True  # 设置为守护线程，主线程结束后子线程自动结束
            th2.start()  # 开启动作线程
            i = i + 1  # 防止重复开启
        else:
            pass
        try:
            if kws.wakeup():  # 检测到唤醒词
                m[0] = 1  # 动作状态1：举手，点阵状态1：唤醒
                speech.play_audio(wakeup_audio_path)  # 唤醒播放
                asr_result = asr.asr()  # 开启录音识别
                print('asr_result:', asr_result)
                m[0] = 2  # 动作状态2：挠头思考，点阵状态2：思考
                if asr_result:
                    # 将识别结果传给智能体让他来回答
                    params = {"model": 'qwen-max',
                              "messages": [
                                  {
                                      "role": "user",
                                      "content": asr_result
                                  },
                              ],
                              "extra_body": {
                                  "enable_search": True
                              }
                              }
                    response = client.llm_origin(params).choices[0].message.content.strip()
                    print('llm response:', response)
                    m[0] = 3  # 动作状态3：挥手回答，点阵状态3：眨眼回答
                    tts.tts(response)
                    runner = False
                else:
                    m[0] = 0  # 回归状态1
                    runner = False  # 点阵屏和动作组共用的线程锁（关）
                    speech.play_audio(no_voice_audio_path)
            else:
                m[0] = 0
                runner = False  # 线程锁（关）
                time.sleep(0.02)
        except KeyboardInterrupt:
            runner = False  # 线程锁（关）
            kws.exit()
            try:
                os.system('pinctrl FAN_PWM a0')
            except:
                pass
            break
        except BaseException as e:
            runner = False  # 线程锁（关）
            print(e)
```
## 5. 对原有功能的影响
两个新增功能是同时运行的在运行时并不会对原功能造成影响，但是退出后最好用上位机将右臂张开，再重新启开机复位，否则有可能会损害电机！点阵屏和电机不会相互阻塞。可以同时进行


## 6. 测试用例
**注：回复、点阵屏显示与动作组同时进行**
-用VNC远程桌面连接树莓派，打开终端,输入：
```
sudo systemctl stop tonypi.service
cd large_models/
python3 interaciton_demo.py
```
测试：

-当机器人初始化时，机器人右手抬到胸前，让点阵屏正对前方，点阵屏上显示当前北京时间（没有冒号）

-当唤醒对机器人说：“小幻小幻”时，机器人回复“我在”，并将左手举起，点阵屏上播放唤醒动画

-当唤醒机器人后对他提问："请介绍一下点阵模块，不超过50字"，机器人挠头，点阵屏上播放思考动画

-过了一会儿，机器人挥手讲解："点阵模块是一种用于显示字符、图形等信息的电子组件，由多个发光二极管按矩阵排列组成"，点阵屏上播放眨眼动画

-想要结束运行，按Ctrl+C，若无效，请多次按下Ctrl+C，不要关闭终端，打开TonyPi上位机，将ID=15的电机调到437-520，然后在终端输入：
```
sudo systemctl restart tonypi.service
```
-即可复位
## 7. 回滚方案
删除第12-15、39-123、127-128、131-144、147、151、167、169、171-172、175-176、179、187行代码


> 开发：goldcoast2 | 日期：2026-02-27