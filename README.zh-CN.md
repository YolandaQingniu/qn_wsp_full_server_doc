# WSP服务器SDK

[English](./README.en.md) | [简体中文](./README.zh-CN.md)


本SDK通过Docker发布.

Docker是开源的容器平台.

它让开发者可以打包应用进容器.

容器简化了应用的分发, 避免分发方需要开发不同编程语言的SDK.

### 角色
本SDK存在4类角色
1. Scale
2. App
3. Docker
4. Server

注: 本文档中的Docker既指容器技术, 也指代本SDK

说明:
+ Docker是夹在Scale和Server间的计算服务器
+ Scale只同APP和Docker进行通讯
+ APP只同Scale和Server通讯
+ Server只同APP和Docker通讯
+ Docker只同Scale和Server通讯

### 请求流程
1.App <----> Scale
+ APP通过蓝牙发送数据到Scale, 告知Scale Docker的域名.
+ Scale通过蓝牙发送数据到APP.

2.Scale ----> Docker
+ 秤通过HTTP请求发送加密数据到Docker. 
+ Docker解密并解析数据, 然后发回响应给秤.

3.Docker ----> Server
+ Docker通过HTTP请求发送数据到Server.
+ Server接收到Docker的数据.

4.Server <----> App
+ APP通过HTTP请求发送数据到Server.
+ Server响应APP请求.

### 如何创建Docker服务?
1. 安装Docker. 参考https://www.docker.com/get-started

2. 创建一个目录并进入目录.
`mkdir youlanda_wsp_full && cd yolanda_wsp_full

3. vim Dockerfile
```Dockerfile
FROM registry.cn-shenzhen.aliyuncs.com/yolanda_open/wsp-full:v1.0.0
ENV CLIENT_URL="https://your-business-server.com"
ENV CLIENT_ID="A_CLIENT_ID_FROM_YOLANDA_PLEASE_CONTACT_US"
# 16位数字或字母的秤端密钥, 需与APP配置一致
ENV SCALE_KEY="1234567812345678"
ENV TZ="Asia/Shanghai"
```

> 贴士: 默认时区是UTC, 可以通过环境变量`TZ`进行更改. 容器日志存储在 log/access.log, 按天进行切割, 并保留7天日志. 容器内端口号为8080, 外部访问端口号可在启动Docker时自行配置.

4. 构建并运行.
docker build . -t yolanda_wsp_full
docker run -p 8080:8080 yolanda_wsp_full

5. 映射应用为域名
示例: 0.0.0.0:8080 映射为 https://your-docker-server.com 或 http://your-docker-server.com

## 接口(Scale <---> Docker)
#### 接口
```text
GET https://your-docker-server.com/yolanda/wsp?code={encrypted_data}
```

## 接口(Docker <---> Server)
### 获取用户信息(Docker -> Server)
#### 接口
```text
POST https://your-business-server.com/wsp/list_users
```

#### 请求参数
| 字段名 | 类型   | 是否必填 | 介绍 | 额外说明                |
| ------ | ------ | -------- | ---- | ----------------------- |
| mac    | String | Y        | Mac  | mock: 12:34:56:78:9A:BC |

#### 请求参数示例
```json
{
    "mac": "12:34:56:78:9A:BC"
}
```

#### 返回参数
| 字段名       | 类型    | 是否必填 | 介绍                 | 额外说明         |
| ------------ | ------- | -------- | -------------------- | ---------------- |
| users        | array   | Y        | 用户                 | 用户列表         |
| - user_index | integer | Y        | 位置                 | mock: 1          |
| - gender     | integer | Y        | 性别                 | mock: 1          |
| - height     | number  | Y        | 身高(cm)             | mock: 180.5      |
| - birthday   | string  | Y        | 生日                 | mock: 1995-06-30 |
| - weight     | number  | Y        | 用户最近一条体重(kg)   | mock: 78.6       |
| - goal_weight| number  | Y        | 用户的目标体重(kg)     | mock: 70.0       |

#### 返回参数示例
```json
{
    "users": [{
        "user_index": 1,
        "gender": 1,
        "height": 180.5,
        "birthday": "1995-06-30",
        "weight": 78.6,
        "goal_weight": 70.0
    },{
        "user_index": 2,
        "gender": 0,
        "height": 168.5,
        "birthday": "1994-02-28",
        "weight": 54.0,
        "goal_weight": 60.0
    },{
        "user_index": 8,
        "gender": 1,
        "height": 174.3,
        "birthday": "1999-12-08",
        "weight": 67.0,
        "goal_weight": 65.0
    }]
}
```

### 上传数据(Docker -> Server)
#### 接口
```text
POST https://your-business-server.com/wsp/create_measurements
```

#### 请求参数

+ 若只有一条记录表示能准确识别用户
+ 若存在多条记录表示不能准确识别用户, 多个用户声明对数据拥有归属权.
+ 确认归属权机制建议采用: APP中弹出页面, 让用户选择数据是否他测量的.

| 字段名          | 类型    | 是否必填 | 介绍               | 额外说明                     |
| --------------- | ------- | -------- | ------------------ | ---------------------------- |
| mac             | string  | Y        | Mac                | mock: 12:34:56:78:9A:BC      |
| records         | array   | Y        | 记录列表           |                              |
| - mac           | string  | Y        | Mac                | mock: 12:34:56:78:9A:BC      |
| - model_id      | string  | Y        | 设备型号 (4 chars) | mock: 0E2B                   |
| - user_index    | integer | Y        | 用户索引           | mock: 1                      |
| - timestamp     | integer | Y        | 测量时间戳 (s)     | mock: 1582698882             |
| - weight        | number  | Y        | 体重 (kg)          | mock: 55.0                   |
| - heart_rate    | integer | Y        | 心率 (BPM)         | mock: 70                     |
| - hmac          | string  | Y        | 签名               | mock: 183476B32E22B26989A... |
| - bmi           | number  | Y        | BMI                | mock：20.1                   |
| - bodyfat       | number  | Y        | 体脂率             | mock：14                     |
| - lbm           | number  | Y        | 去脂体重           | mock：50.1                   |
| - subfat        | number  | Y        | 皮下脂肪           | mock：12.7                   |
| - visfat        | number  | Y        | 内脏脂肪           | mock：3.46                   |
| - water         | number  | Y        | 水分               | mock：62.2                   |
| - bmr           | integer | Y        | 基础代谢           | mock：1451                   |
| - muscle        | number  | Y        | 骨骼肌率           | mock：55.6                   |
| - sinew         | number  | Y        | 肌肉量             | mock：47.5                   |
| - bone          | number  | Y        | 骨量               | mock：2.51                   |
| - protein       | number  | Y        | 蛋白质             | mock：19.5                   |
| - score         | number  | Y        | 分数               | mock：90.2                   |
| - body_age      | integer | Y        | 体年龄             | mock：20                     |
| - body_shape    | integer | Y        | 体型               | mock：4                      |
| - cardiac_index | number  | Y        | 心脏指数           | mock：0                      |

#### 请求参数示例
```json
{
  "mac": "12:34:56:78:9A:BC",
  "records": [{
      "mac": "12:34:56:78:9A:BC",
      "model_id": "0E2B",
      "user_index": 1,
      "timestamp": 1527855059,
      "weight": 55.0,
      "heart_rate": 70,
      "hmac": "70D57038CC39FEA98FBB9A4132B35422752520A027E97ADAE55BD57F9A6EB13A27AD4447D806316227EF96A5979C6F529D15DAF42D7ADD17436739ABE38820CEBF15D58ABAB1F30DAA1EC67362F463A412AAB8BCBA80A594F0B194C197B4C1620BFCF4F7BA5A3743825A7156AEB1F575E22CA4E5BED9237DF838365DFD3E88BA74EDB0AD7F0809DF7DCF4C2F3EA67387C8EE85C25E83044940675FB31BF693239A37E23247F60E651ABD3885BBEAE29CC8EE85C25E83044940675FB31BF69323DA10A0956A1709A0E895FA7740EEB9A1AC7AC7A1F0A2D8365BD261D0FA6C50398A2AF8A4D7003A75E5178E2B52A7502BAE53A619BDE39B1C8098C6A00D4220AF589D389ABEE16D80F5C7E2109E79B95BC7AE52DD296697750F6484B0AB29AD92B5FCABF2E71CDB5F15D89A04A5B01EE13EB4C3F4DB748ACFFBDF4F227478ED69A59A1592642C7420F52D6787F337ADAE9106A14693CEC5AD814D14CEACFC8005BD39289E27ECE343F4B0CF99BCB4DBC950F76FEFB0867ED0932DE5635872AA65CA97902A2898C323861883E1B51E9F5B",
      "bmi": 20.1,
      "bodyfat": 14,
      "lbm": 50.1,
      "subfat": 12.7,
      "visfat": 3.46,
      "water": 62.2,
      "bmr": 1451,
      "muscle": 55.6,
      "sinew": 47.5,
      "bone": 2.51,
      "protein": 19.5,
      "score": 90.2,
      "body_age": 20,
      "body_shape": 4,
      "cardiac_index": 0
  }]
}
```

#### 返回参数
| 字段名     | 类型 | 是否必填 | 介绍     | 额外说明   |
| ---------- | ---- | -------- | -------- | ---------- |
| is_success | bool | Y        | 是否成功 | mock: true |

#### 返回参数示例
```json
{
   "is_success": true
}
```

### 上传八电极数据(Docker -> Server)
#### 接口
```text
POST https://your-business-server.com/wsp/create_eight_measurements
```

#### 请求参数

+ 若只有一条记录表示能准确识别用户
+ 若存在多条记录表示不能准确识别用户, 多个用户声明对数据拥有归属权.
+ 确认归属权机制建议采用: APP中弹出页面, 让用户选择数据是否他测量的.

| 字段名                    | 类型    | 是否必填 | 介绍                | 额外说明                     |
| ------------------------- | ------- | -------- | ------------------- | ---------------------------- |
| mac                       | string  | Y        | Mac                 | mock: 12:34:56:78:9A:BC      |
| records                   | array   | Y        | 记录列表            |                              |
| - mac                     | string  | Y        | Mac                 | mock: 12:34:56:78:9A:BC      |
| - model_id                | string  | Y        | 设备型号 (4 chars)  | mock: 0E2B                   |
| - user_index              | integer | Y        | 用户索引            | mock: 1                      |
| - timestamp               | integer | Y        | 测量时间戳 (s)      | mock: 1582698882             |
| - weight                  | number  | Y        | 体重 (kg)           | mock: 55.0                   |
| - heart_rate              | integer | Y        | 心率 (BPM)          | mock: 70                     |
| - hmac                    | string  | Y        | 签名                | mock: 183476B32E22B26989A... |
| - bmi                     | number  | Y        | BMI                 | mock：20.1                   |
| - bodyfat                 | number  | Y        | 体脂率              | mock：14                     |
| - lbm                     | number  | Y        | 去脂体重            | mock：50.1                   |
| - subfat                  | number  | Y        | 皮下脂肪            | mock：12.7                   |
| - visfat                  | number  | Y        | 内脏脂肪            | mock：3.46                   |
| - water                   | number  | Y        | 水分                | mock：62.2                   |
| - bmr                     | integer | Y        | 基础代谢            | mock：1451                   |
| - muscle                  | number  | Y        | 骨骼肌率            | mock：55.6                   |
| - sinew                   | number  | Y        | 肌肉量              | mock：47.5                   |
| - bone                    | number  | Y        | 骨量                | mock：2.51                   |
| - protein                 | number  | Y        | 蛋白质              | mock：19.5                   |
| - score                   | number  | Y        | 分数                | mock：90.2                   |
| - body_age                | integer | Y        | 体年龄              | mock：20                     |
| - body_shape              | integer | Y        | 体型                | mock：4                      |
| - cardiac_index           | number  | Y        | 心脏指数            | mock：0                      |
| - right_arm_muscle_weight | number  | Y        | 肌肉量 (右上肢)     | mock：0.8                    |
| - left_arm_muscle_weight  | number  | Y        | 肌肉量 (左上肢)     | mock：0.8                    |
| - right_leg_muscle_weight | number  | Y        | 肌肉量 (右下肢)     | mock：0.8                    |
| - left_leg_muscle_weight  | number  | Y        | 肌肉量 (左下肢)     | mock：0.8                    |
| - trunk_muscle_weight     | number  | Y        | 肌肉量 (躯干)       | mock：0.8                    |
| - right_arm_fat           | number  | Y        | 脂肪百分比 (右上肢) | mock：0.8                    |
| - left_arm_fat            | number  | Y        | 脂肪百分比 (左上肢) | mock：0.8                    |
| - right_leg_fat           | number  | Y        | 脂肪百分比 (右下肢) | mock：0.8                    |
| - left_leg_fat            | number  | Y        | 脂肪百分比 (左下肢) | mock：0.8                    |
| - trunk_fat               | number  | Y        | 脂肪百分比 (躯干)   | mock：0.8                    |

#### 请求参数示例
```json
{
  "mac": "12:34:56:78:9A:BC",
  "records": [{
      "mac": "12:34:56:78:9A:BC",
      "model_id": "0E2B",
      "user_index": 1,
      "timestamp": 1527855059,
      "weight": 55.0,
      "heart_rate": 70,
      "hmac": "70D57038CC39FEA98FBB9A4132B35422752520A027E97ADAE55BD57F9A6EB13A27AD4447D806316227EF96A5979C6F529D15DAF42D7ADD17436739ABE38820CEBF15D58ABAB1F30DAA1EC67362F463A412AAB8BCBA80A594F0B194C197B4C1620BFCF4F7BA5A3743825A7156AEB1F575E22CA4E5BED9237DF838365DFD3E88BA74EDB0AD7F0809DF7DCF4C2F3EA67387C8EE85C25E83044940675FB31BF693239A37E23247F60E651ABD3885BBEAE29CC8EE85C25E83044940675FB31BF69323DA10A0956A1709A0E895FA7740EEB9A1AC7AC7A1F0A2D8365BD261D0FA6C50398A2AF8A4D7003A75E5178E2B52A7502BAE53A619BDE39B1C8098C6A00D4220AF589D389ABEE16D80F5C7E2109E79B95BC7AE52DD296697750F6484B0AB29AD92B5FCABF2E71CDB5F15D89A04A5B01EE13EB4C3F4DB748ACFFBDF4F227478ED69A59A1592642C7420F52D6787F337ADAE9106A14693CEC5AD814D14CEACFC8005BD39289E27ECE343F4B0CF99BCB4DBC950F76FEFB0867ED0932DE5635872AA65CA97902A2898C323861883E1B51E9F5B",
      "bmi": 20.1,
      "bodyfat": 14,
      "lbm": 50.1,
      "subfat": 12.7,
      "visfat": 3.46,
      "water": 62.2,
      "bmr": 1451,
      "muscle": 55.6,
      "sinew": 47.5,
      "bone": 2.51,
      "protein": 19.5,
      "score": 90.2,
      "body_age": 20,
      "body_shape": 4,
      "cardiac_index": 0,
      "right_arm_muscle_weight": 2.83209,
      "left_arm_muscle_weight": 2.76834,
      "right_leg_muscle_weight": 8.72593,
      "left_leg_muscle_weight": 8.68123,
      "trunk_muscle_weight": 23.85385,
      "right_arm_fat": 2.18414,
      "left_arm_fat": 2.18987,
      "right_leg_fat": 4.97996,
      "left_leg_fat": 4.96095,
      "trunk_fat": 16.16568,
  }]
}
```

#### 返回参数
| 字段名     | 类型 | 是否必填 | 介绍     | 额外说明   |
| ---------- | ---- | -------- | -------- | ---------- |
| is_success | bool | Y        | 是否成功 | mock: true |

#### 返回参数示例
```json
{
   "is_success": true
}
```

### APP与Server接口交互
APP与Scale的交互需要同步用户数据, 故APP需从Server获取用户数据列表, 也会创建用户数据.

Server需要维护用户数据的位置、性别、生日、身高、最近一次体重.

在更新用户, 删除用户, 解绑设备时, Server都需维护用户信息的一致性.
如:
+ 更新用户时: 更新用户
+ 删除用户时: 删除用户
+ 解绑设备时: 删除用户

#### 伪代码(宿主语言: ruby)
```ruby
# File: config/routers.rb
Rails.application.routers.draw do
  namespace "wsps", format: "json" do
    post "create_user"
    get "list_users"
  end
end

# File: app/controllers/wsps_controller.rb
class WspsController < ApplicationController
  # POST https://your-business-server.com/wsps/create_user
  # 创建用户
  def create_user
    User.create(
      mac: params["mac"],
      user_index: params["user_index"],
      gender: params["gender"],
      birthday: params["birthday"],
      height: params["height"],
      weight: params["weight"]
    )
    render json: {}
  end

  # GET https://your-business-server.com/wsps/list_users
  # 获取用户列表
  def list_users
    users = User.where(mac: params["mac"])
    user_json = users.map do |user|
      {
        user_index: user.user_index.to_i,
        gender: user.gender.to_i,
        birthday: user.birthday.to_s,
        height: user.height.to_f,
        weight: user.weight.to_f
      }
    end
    render json: user_json
  end
end
```
