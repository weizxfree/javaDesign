
# 8113版本项目总结  #  

|  日期  | 版本  | 说明  | 作者  
|--|--|--|--|
|  2018/8/14| V1.0 | 项目总结|韦章翔
  
# 1. 总览 #  
  
### 1.1 背景介绍 ###  
晓译翻译机2代8113版本发布期间，整个版本质量比较差，交付能力较前版本明显减弱。具体表现：交付延时，bug反复打回，性能问题严重，代码提交松散。
  
# 2. 总结 #

# 组件图
![组件图](https://github.com/weizxfree/javaDesign/blob/master/doc/offline.png?raw=true)
## 时序图

 1. 语种切换

```mermaid
sequenceDiagram
User->> App: 启动app
App->> App: start Offline Service 
Note right of App: service启动时会初始化init esr trs tts
opt      当前选择语种支持离线
        App->>App: esr preload
        App->>App: trs preload
end
User->> App: 手动切换语种/场景
Note right of User: 切换场景指按键和拍照之间相互切换
opt      待切换语种支持离线且语种变更
        App->>App: esr preload
        App->>App: trs preload
        Note over App: trs load 有15s超时处理
end
App->>App: tts 引擎load 
Note right of App: tts引擎load资源很快，没有preload，只在合成前load
```

 2. 按键翻译
```mermaid
sequenceDiagram
User->> App: 中/英键按下
Note over App: 网络未连接
  alt trs load success
        App-xOfflineService: start recognize
    else trs load not success
         alt trs is loading
         App->>App: toast 提示
		 else trs load fail
		 App->>App: reload、toast 提示
		 end
    end
User->> App: 中/英键按起
App->> App: 停止识别 stop recognize
OfflineService--xApp: 识别结果
Note right of App: 检查trs是否已加载,没有加载的话切换引擎再次加载，流程中断
App-xOfflineService: start trs
Note right of App: 设置30s翻译超时
OfflineService--xApp: 翻译结果
App->> App: 翻译记录上屏
Note over App: 临时根据语种加载tts资源
App-xOfflineService: start tts

```
3. esr trs load unload 实现
通过handler维护队列，如果队列中存在未执行的load或start 任务，则直接移除；
添加最新的任务，保障频繁切换语种情况下，不需要排队等待。


## 离线翻译Tag 
 
 - EsrEngine/QRecognizeEngine
 - TransEngine/QTranslateEngine
 - FlyTtsWrapper （中英）/QTtsEngine
 - RusTtsWrapper （俄，日，韩）/QTtsEngine

## 离线翻译资源地址
**esr** 
> /sdcard/esr/*

**trs**  
> /sdcard/itrans_db3300
> /sdcard/NiuTransTransformer

**tts**
>  /sdcard/tts/*

## 离线翻译log地址

>中英： /data/data/com.iflytek.android.device/files/log_cnen.log
俄日韩： /data/data/com.iflytek.android.device/files/log_ru.log

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDczNDYzODM3XX0=
-->