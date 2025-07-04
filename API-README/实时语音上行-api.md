# 扣子websocketapi

除了基于 WebRTC 的实时通信方案外，扣子智能语音还推出了 WebSocket OpenAPI 方案，用于实现用户与智能体之间的实时语音通话。该方案通过 WebSocket 协议提供高效、灵活的语音交互能力，适用于多种应用场景。 

## 注意事项 

WebSocket OpenAPI 支持的音频编码格式如下： 

* 上行：输入音频支持 PCM、G711A、G711U，上传语音文件支持 WAV 或 OGG 格式，默认格式为 WAV。 
* 下行：输出音频支持 PCM 或 OPUS 格式，默认为采样率 24000 的 PCM 片段。 

## 准备工作 

在开始集成 WebSocket OpenAPI 之前，你需要先完成以下准备工作。

| 操作 | 说明 |
|------|------|
| 发布智能体 | 已成功搭建并发布智能体为 API 服务。搭建步骤请参见搭建一个 AI 助手智能体，发布步骤请参见发布智能体为 API 服务。 |
| 获取访问密钥 | 获取访问密钥，用于身份认证与鉴权。<br>* 体验或调试场景：建议生成短期的个人访问令牌（PAT），以快速完成 Realtime SDK 的整体流程。个人访问令牌的获取方法请参见添加个人访问令牌。 |

## 实现语音通话 

### 步骤一：建立 WebSocket 连接 

发起 HTTP 请求时，在请求头（Header）中添加Authorization信息。Authorization的取值固定为Bearer $Access_Token，用于扣子 OpenAPI 鉴权的访问密钥。将您在准备工作中获取的访问密钥替换掉 $Access_Token 后再发起请求。

| 操作 | 说明 |
|------|------|
| 发布智能体 | 已成功搭建并发布智能体为 API 服务。搭建步骤请参见搭建一个 AI 助手智能体，发布步骤请参见发布智能体为 API 服务。 |
| 获取访问密钥 | 获取访问密钥，用于身份认证与鉴权。<br>* 体验或调试场景：建议生成短期的个人访问令牌（PAT），以快速完成 Realtime SDK 的整体流程。个人访问令牌的获取方法请参见添加个人访问令牌。<br>* 线上环境：在线上环境中，应使用 OAuth 鉴权方案。OAuth 鉴权方案的详细说明请参见OAuth 应用管理。 |

```javascript
import WebSocket from 'ws'; 
 
const url = `wss://ws.coze.cn/v1/audio/transcriptions?authorization=Bearer pat_OYDacMzM3WyOWV3Dtj2bHRMymzxP****`; 
# 如果是流式语音对话接口，需要在 url 中带上 bot_id  
# const url = wss://ws.coze.cn/v1/chat?bot_id=73791654286875***&authorization=Bearer pat_OYDacMzM3WyOWV3Dtj2bHRMymzxP****`; 
const ws = new WebSocket(url); 
 
ws.on('open', function open() { 
  console.log('Connected to server.'); 
}); 
 
ws.on('message', function incoming(message) { 
  console.log(JSON.parse(message.toString())); 
}); 
```

### 步骤二：发送和接收事件 

通过 WebSocket 与扣子智能体进行实时语音交互时，需要通过 WebSocket 接口发送和接收消息。成功连接后，客户端可以发送和接收代表文本、音频、配置更新等的事件。客户端可以发送的事件消息以及从服务器接收的事件消息列表请参见双向流式对话事件。 

发送和接收事件的示例代码如下：

```python
# To send a client event, serialize a dictionary to JSON 
# of the proper event type 
def on_open(ws): 
    print("Connected to server.") 
     
    data = { 
        "role": "user", 
        "content_type": "text", 
        "content": "你好呀" 
    } 
    event = { 
        "event_type": "conversation.message.create", 
        "data":  data 
    } 
    ws.send(json.dumps(event)) 
 
# Receiving messages will require parsing message payloads 
# from JSON 
def on_message(ws, message): 
    data = json.loads(message) 
    print("Received event:", json.dumps(data, indent=2)) 
```

## 事件类型 

智能语音 WebSocket 事件包括上行事件和下行事件。每个事件有 ID 和 EventType，通过 EventType 可以区分具体的事件类型，每个事件类型对应的 Payload 在 Data 中，开发者可以按需去提取需要的内容。 

* 上行事件：设备端上报给服务端的事件。应用程序需要根据扣子平台提供的事件结构，在触发事件时填充字段内容并上报事件。 
* 下行事件：服务端下发给设备端的事件，应用程序需要解析下行事件，并根据业务需求进行下一步操作。 

### 注意事项 

* 每个上行事件 ID 建议不要重复，故障排查场景下便于定位问题。 
* 每个事件有 ID 和 EventType，通过 EventType 可以区分具体的事件类型，每个事件类型对应的 Payload 在 Data 中，开发者可以按需去提取需要的内容。 

## 公共参数 

智能语音 WebSocket 事件的公共参数如下：

| 参数名称 | 类型 | 描述 |
|---------|-----|------|
| id | String | 事件 ID，也就是事件的唯一标识。由客户端或服务端生成，在故障排查场景下用于定位具体的事件，便于排查问题。 |
| event_type | String | 事件的类型。 |
| data | JSON | 事件的详细信息，其中包含具体事件的业务字段。 |

## 上行

扣子websocketapi-上行

扣子提供流式语音对话 WebSocket OpenAPI，向指定的智能体发起语音对话。 
双向流式语音对话场景下的各类事件详细信息可参考双向流式对话事件。 

### 接口信息 

| 项目 | 说明 |
|------|------|
| URL | wss://ws.coze.cn/v1/chat |
| Headers | Authorization Bearer $Access_Token<br>用于验证客户端身份的访问令牌。你可以在扣子平台中生成访问令牌，详细信息请参考准备工作。 |
| 权限 | chat |
| 接口说明 | 向指定的智能体发起语音对话。 |

### Query参数

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| bot_id | String | 必选 | 需要关联的智能体 ID。<br>进入智能体的开发页面，开发页面 URL 中 bot 参数后的数字就是智能体 ID。例如 https://www.coze.com/space/341****/bot/73428668*****，Bot ID 为 73428668*****。 |
| workflow_id | String | 可选 | 待执行的对话流 ID，此对话流应已发布。<br>进入对话流编排页面，在页面 URL 中，workflow 参数后的数字就是对话流 ID。例如 https://www.coze.com/work_flow?space_id=42463***&workflow_id=73505836754923***，对话流 ID 为 73505836754923***。 |

### 建连示例代码

```javascript
import WebSocket from 'ws'; 
 
const url = `wss://ws.coze.cn/v1/chat?bot_id=${BOT_ID}&authorization=Bearer ${ACCESS_TOKEN}`; 
const ws = new WebSocket(url); 
 
ws.on('open', function open() { 
  console.log('Connected to server.'); 
}); 
 
ws.on('message', function incoming(message) { 
  console.log(JSON.parse(message.toString())); 
}); 
```

## 上行事件 

### 更新对话配置 

* 事件类型：chat.update
* 事件说明：此事件可以更新当前对话连接的配置项，若更新成功，会收到 chat.updated 的下行事件，否则，会收到 error 下行事件。 
* 事件结构：

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| id | String | 必选 | 客户端自行生成的事件 ID，方便定位问题。 |
| event_type | String | 必选 | 固定为 chat.update。 |
| data | Object | 可选 | 事件数据，包含对话配置的详细信息。 |
| data.chat_config | Object | 可选 | 对话配置。 |
| data.chat_config.meta_data | Map<String, String> | 可选 | 附加信息，通常用于封装一些业务相关的字段。查看对话消息详情时，系统会透传此附加信息。自定义键值对，应指定为 Map 对象格式。长度为 16 对键值对，其中键（key）的长度范围为 1～64 个字符，值（value）的长度范围为 1～512 个字符。 |
| data.chat_config.custom_variables | Map<String, String> | 可选 | 智能体中定义的变量。在智能体 prompt 中设置变量 {{key}} 后，可以通过该参数传入变量值，同时支持 Jinja2 语法。详细说明可参考变量示例。变量名只支持英文字母和下划线。 |
| data.chat_config.extra_params | Map<String, String> | 可选 | 附加参数，通常用于特殊场景下指定一些必要参数供模型判断，例如指定经纬度，并询问智能体此位置的天气。自定义键值对格式，其中键（key）仅支持设置为：<br>* latitude（纬度，此时值（Value）为纬度值，例如 39.9800718）。<br>* longitude（经度，此时值（Value）为经度值，例如 116.309314）。 |
| data.chat_config.user_id | String | 可选 | 标识当前与智能体的用户，由使用方自行定义、生成与维护。user_id 用于标识对话中的不同用户，不同的 user_id，其对话的上下文消息、数据库等对话记忆数据互相隔离。如果不需要用户数据隔离，可将此参数固定为一个任意字符串，例如 123，abc 等。 |
| data.chat_config.conversation_id | String | 可选 | 标识对话发生在哪一次会话中。会话是智能体和用户之间的一段问答交互。一个会话包含一条或多条消息。对话是会话中对智能体的一次调用，智能体会将对话中产生的消息添加到会话中。可以使用已创建的会话，会话中已存在的消息将作为上下文传递给模型。创建会话的方式可参考创建会话。对于一问一答等不需要区分 conversation 的场合可不传该参数，系统会自动生成一个会话。不传的话会默认创建一个新的 conversation。 |
| data.chat_config.auto_save_history | Boolean | 可选 | 是否保存本次对话记录。<br>* true：（默认）会话中保存本次对话记录，包括本次对话的模型回复结果、模型执行中间结果。<br>* false：会话中不保存本次对话记录，后续也无法通过任何方式查看本次对话信息、消息详情。在同一个会话中再次发起对话时，本次会话也不会作为上下文传递给模型。 |
| data.chat_config.parameters | Map<String, any> | 可选 | 设置对话流的输入参数。<br>* 对话流的输入参数 USER_INPUT 应在 additional_messages 中传入，在 parameters 中的 USER_INPUT 不生效。<br>* 如果 parameters 中未指定 CONVERSATION_NAME 或其他输入参数，则使用参数默认值运行对话流；如果指定了这些参数，则使用指定值。 |
| data.input_audio | Object | 可选 | 输入音频格式。 |
| data.input_audio.format | String | 可选 | 输入音频的格式，支持 pcm、wav、ogg。默认为 wav。 |
| data.input_audio.codec | String | 可选 | 输入音频的编码，支持 pcm、opus、g711a、g711u。默认为 pcm。<br>如果音频编码格式为 g711a 或 g711u，format 请设置为 pcm。 |
| data.input_audio.sample_rate | Integer | 可选 | 输入音频的采样率，默认是 24000。支持 8000、16000、22050、24000、32000、44100、48000。<br>如果音频编码格式 codec 为 g711a 或 g711u，音频采样率需设置为 8000。 |
| data.input_audio.channel | Integer | 可选 | 输入音频的声道数，支持 1（单声道）、2（双声道）。默认是 1（单声道）。 |
| data.input_audio.bit_depth | Integer | 可选 | 输入音频的位深，默认是 16，支持8、16和24。 |
| data.output_audio | Object | 可选 | 输出音频格式。 |
| data.output_audio.codec | String | 可选 | 输出音频编码，支持 pcm、opus。默认是 pcm。 |
| data.output_audio.pcm_config | Object | 可选 | 当 codec 设置为 opus 时，不需要设置此字段。<br>当 codec 设置为 pcm 时，返回的 PCM 数据将固定为单声道，采样深度为 16 位。 |
| data.output_audio.pcm_config.sample_rate | Integer | 可选 | 输出 pcm 音频的采样率，默认是 24000。支持 8000、16000、22050、24000、32000、44100、48000。 |
| data.output_audio.pcm_config.frame_size_ms | Float | 可选 | 输出每个 pcm 包的时长，单位 ms，默认不限制。 |
| data.output_audio.pcm_config.limit_config | Object | 可选 | 输出音频限流配置，默认不限制。 |
| data.output_audio.pcm_config.limit_config.period | Integer | 可选 | 周期的时长，单位为秒。例如设置为 10 秒，则以 10 秒作为一个周期。 |
| data.output_audio.pcm_config.limit_config.max_frame_num | Integer | 可选 | 周期内，最大返回 pcm 包数量。 |
| data.output_audio.opus_config | Object | 可选 | 当 codec 设置为 pcm 时，不需要设置此字段。 |
| data.output_audio.opus_config.bitrate | Integer | 可选 | 输出 opus 的码率，默认 48000。 |
| data.output_audio.opus_config.use_cbr | Boolean | 可选 | 输出 opus 是否使用 CBR 编码，默认为 false。 |
| data.output_audio.opus_config.frame_size_ms | Float | 可选 | 输出 opus 的帧长，默认是 10。可选值：<br>2.5、5、10、20、40、60 |
| data.output_audio.opus_config.limit_config | Object | 可选 | 输出音频限流配置，默认不限速。 |
| data.output_audio.opus_config.limit_config.period | Integer | 可选 | 周期的时长，单位为秒。例如设置为 10 秒，则以 10 秒作为一个周期。 |
| data.output_audio.opus_config.limit_config.max_frame_num | Integer | 可选 | 周期内最大返回的 Opus 帧数量。 |
| data.output_audio.speech_rate | Integer | 可选 | 输出音频的语速，取值范围 [-50, 100]，默认为 0。-50 表示 0.5 倍速，100 表示 2 倍速。 |
| data.output_audio.voice_id | String | 可选 | 输出音频的音色 ID，默认是柔美女友音色。你可以调用查看音色列表 API 查看当前可用的所有音色 ID。 |
| data.event_subscriptions | Array<String> | 可选 | 需要订阅下行事件的事件类型列表。不设置或者设置为空为订阅所有下行事件。 |
| data.need_play_prologue | Boolean | 可选 | 是否需要播放开场白，默认为 false。 |
| data.prologue_content | String | 可选 | 自定义开场白，need_play_prologue 设置为 true 时生效。如果不设定自定义开场白则使用智能体上设置的开场白。 |
| data.turn_detection | Object | 可选 | 转检测配置。 |
| data.turn_detection.type | String | 可选 | 用户演讲检测模式，包括：<br>* server_vad ：语音数据会传输到服务器端进行实时分析，服务器端的语音活动检测算法会判断用户是否在说话。<br>* client_interrupt：（默认）客户端实时分析语音数据，并检测用户是否已停止说话。 |
| data.turn_detection.prefix_padding_ms | Integer | 可选 | server_vad 模式下，VAD 检测到语音之前要包含的音频量，单位为 ms。默认为 600ms。 |
| data.turn_detection.silence_duration_ms | Integer | 可选 | server_vad 模式下，检测语音停止的静音持续时间，单位为 ms。默认为 500ms。 |
| data.turn_detection.interrupt_config | Object | 可选 | server_vad 模式下打断策略配置 |
| data.turn_detection.interrupt_config.mode | String | 可选 | 打断模式, 包括:<br>* keyword_contains模式下，说话内容包含关键词才会打断模型回复。例如关键词"扣子"，用户正在说"你好呀扣子......" / "扣子你好呀"，模型回复都会被打断。<br>* keyword_prefix模式下，说话内容前缀匹配关键词才会打断模型回复。例如关键词"扣子"，用户正在说"扣子你好呀......"，模型回复就会被打断，而用户说"你好呀扣子......"，模型回复不会被打断。 |
| data.turn_detection.interrupt_config.keywords | Array<String> | 可选 | 打断的关键词配置，最多同时限制 5 个关键词，每个关键词限定长度在6-24个字节以内(2-8个汉字以内), 不能有标点符号。 |
| data.asr_config | Object | 可选 | 语音识别配置，包括热词和上下文信息，以便优化语音识别的准确性和相关性。 |
| data.asr_config.hot_words | Array<String> | 可选 | 请输入热词列表，以便提升这些词汇的识别准确率。<br>所有热词加起来最多100个 Tokens，超出部分将自动截断。 |
| data.asr_config.context | String | 可选 | 请输入上下文信息。<br>最多输入 800 个 Tokens，超出部分将自动截断。 |
| data.asr_config.user_language | String | 可选 | 用户说话的语种，默认为 common。选项包括：<br>* common：大模型语音识别，可自动识别中英粤。<br>* zh：小模型语音识别，中文。<br>* cant：小模型语音识别，粤语。<br>* sc：小模型语音识别，川渝。<br>* en：小模型语音识别，英语。<br>* ja：小模型语音识别，日语。<br>* ko：小模型语音识别，韩语。<br>* fr：小模型语音识别，法语。<br>* id：小模型语音识别，印尼语。<br>* es：小模型语音识别，西班牙语。<br>* pt：小模型语音识别，葡萄牙语。<br>* ms：小模型语音识别，马来语。<br>* ru：小模型语音识别，俄语。 |
| data.asr_config.enable_ddc | Boolean | 可选 | 将语音转为文本时，是否启用语义顺滑。默认为 true。<br>* true：系统在进行语音处理时，会去掉识别结果中诸如 "啊""嗯" 等语气词，使得输出的文本语义更加流畅自然，符合正常的语言表达习惯，尤其适用于对文本质量要求较高的场景，如正式的会议记录、新闻稿件生成等。<br>* false：系统不会对识别结果中的语气词进行处理，识别结果会保留原始的语气词。 |
| data.asr_config.enable_itn | Boolean | 可选 | 将语音转为文本时，是否进行文本纠错，默认为 true。 |
| data.asr_config.enable_punc | Boolean | 可选 | 将语音转为文本时，是否给文本加上标点符号。默认为 true。 |

* 事件示例：

```json
{ 
    "id": "event_id", 
    "event_type": "chat.update", 
    "data": { 
        "chat_config": { 
            "auto_save_history": true, // 保存历史记录。默认 true 
            "conversation_id": "xxxx", // conversation_id 
            "user_id": "xxx",  // 标识当前与智能体的用户，由使用方自行定义、生成与维护。user_id 用于标识对话中的不同用户，不同的 user_id，其对话的上下文消息、数据库等对话记忆数据互相隔离。如果不需要用户数据隔离，可将此参数固定为一个任意字符串 
            "meta_data": {}, // 附加信息,通常用于封装一些业务相关的字段。查看对话消息详情时,系统会透传此附加信息。 
            "custom_variables": {}, // 智能体中定义的变量。在智能体prompt中设置变量{{key}}后,可以通过该参数传入变量值,同时支持Jinja2语法。详细说明可参考变量示例。变量名只支持英文字母和下划线。 
            "extra_params": {},   // 附加参数,通常用于特殊场景下指定一些必要参数供模型判断,例如指定经纬度,并询问智能体此位置的天气。自定义键值对格式,其中键(key)仅支持设置为:latitude:纬度,此时值(Value)为纬度值,例如39.9800718。longitude:经度,此时值(Value)为经度值,例如116.309314。 
            "parameters": {"custom_var_1": "测试"} 
        }, 
        "input_audio": {         // 输入音频格式 
            "format": "pcm",       // 输入音频格式，支持 pcm/wav/ogg。默认 wav 
            "codec": "pcm",         // 输入音频编码。 pcm/opus。默认 pcm 
            "sample_rate": 24000,  // 采样率 
            "channel": 1, // 通道数 
            "bit_depth": 16 // 位深 
        }, 
        "output_audio": {        // 输出音频格式 
            "codec": "pcm",        
            "pcm_config": { 
                "sample_rate": 16000,  // 默认  24000 
                "frame_size_ms": 50, 
                "limit_config": { 
                    "period": 1, 
                    "max_frame_num": 22 
                } 
            }, 
            "speech_rate": 0,  // 回复的语速，取值范围 [-50, 100]，默认为 0，-50 表示 0.5 倍速，100 表示 2倍速 
            "voice_id": "7426720361733046281" 
        } 
    } 
} 
```

### 流式上传音频片段 

* 事件类型：input_audio_buffer.append
* 事件说明：流式向服务端提交音频的片段。 
* 事件结构：

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| id | String | 必选 | 客户端自行生成的事件 ID，方便定位问题。 |
| event_type | String | 必选 | 固定为 input_audio_buffer.append。 |
| data | Object | 必选 | 事件数据，包含音频片段信息。 |
| data.delta | String | 必选 | base64 编码后的音频片段。 |

```json
{ 
  "id": "event_id", 
  "event_type": "input_audio_buffer.append", 
  "data": { 
     "delta": "base64EncodedAudioDelta" 
  } 
}
```

### 提交音频 

* 事件类型：input_audio_buffer.complete
* 事件说明：客户端发送 input_audio_buffer.complete 事件来告诉服务端提交音频缓冲区的数据。服务端提交成功后会返回 input_audio_buffer.completed 事件。在 server_vad 模式下，提交此事件无效。 
* 事件结构：

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| id | String | 必选 | 客户端自行生成的事件 ID，方便定位问题。 |
| event_type | String | 必选 | 固定为 input_audio_buffer.complete。 |

```markdown
{ 
  "id": "event_id", 
  "event_type": "input_audio_buffer.complete" 
} 
```

### 清除缓冲区音频 

* 事件类型：input_audio_buffer.clear
* 事件说明：客户端发送 input_audio_buffer.clear 事件来告诉服务端清除缓冲区的音频数据。服务端清除完后将返回 input_audio_buffer.cleared 事件。在 server_vad 模式下，提交此事件无效。 
* 事件结构：

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| id | String | 必选 | 客户端自行生成的事件 ID，方便定位问题。 |
| event_type | String | 必选 | 固定为 input_audio_buffer.clear。 |

```markdown
{ 
  "id": "event_1", 
  "event_type": "input_audio_buffer.clear" 
}
```

### 清除上下文 

* 事件类型：conversation.clear
* 事件说明：清除上下文，会在当前 conversation 下新增一个 section，服务端处理完后会返回 conversation.cleared 事件。 
* 事件结构:

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| id | String | 必选 | 客户端自行生成的事件 ID，方便定位问题。 |
| event_type | String | 必选 | 固定为 conversation.clear。 |

```markdown
{ 
  "id": "event_1", 
  "event_type": "conversation.clear" 
} 
```

### 打断智能体输出 

* 事件类型：conversation.chat.cancel
* 事件说明：发送此事件可取消正在进行的对话，中断后，服务端将会返回 conversation.chat.canceled 事件。 
* 事件结构：

| 参数 | 类型 | 是否必选 | 说明 |
|------|------|----------|------|
| id | String | 必选 | 客户端自行生成的事件 ID，方便定位问题。 |
| event_type | String | 必选 | 必填 conversation.chat.cancel。 |

```markdown
{ 
  "id": "7446668538246561827", 
  "event_type": "conversation.chat.cancel" 
} 
```

