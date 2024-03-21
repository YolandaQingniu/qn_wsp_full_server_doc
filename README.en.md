# Yolanda WSP Docker SDK Document

[English](./README.en.md) | [简体中文](./README.zh-CN.md)

The SDK is released by Docker.

Docker is an open source containerization platform.

It enables developers to package applications into containers. 

Containers simplify delivery of distributed applications.

### Roles

There are 4 roles in the SDK.

1. Scale
2. App
3. Docker
4. Server

Note: Docker in this document refers to both container technology and this SDK.

Instruction:

+ `Docker` is a computing server sandwiched between `Scale` and `Server`.
+ `Scale` only communicates with `APP` and `Docker`.
+ `APP` only communicates with `Scale` and `Server`.
+ `Server` only communicates with `APP` and `Docker`.
+ `Docker` only communicates with `Scale` and `Server`.

### The Request Flow

1.App <----> Scale

+ `App` sends data to `Scale` via bluetooth, to tell what is the domain of `Docker`.
+ `Scale` sends data to `App` via bluetooth.

2.Scale ----> Docker

+ `Scale` sends encrypted data to `Docker` Server via HTTP request.
+ `Docker` Server decrypts and parses the data, then responses to `Scale`.

3.Docker ----> Server

+ `Docker` sends data to `Server` via HTTP request.
+ `Server` receives the data from `Docker`.

4.Server <----> App

+ `App` sends data to `Server` via HTTP request.
+ `Server` responses to `App`.

### How to create Docker Server?

1. Install Docker. See https://www.docker.com/get-started

2. Make a directory and change into it.
   
```shell
   mkdir youlanda_wsp_full && cd yolanda_wsp_full
```

3. vim Dockerfile

```Dockerfile
FROM registry.cn-shenzhen.aliyuncs.com/yolanda_open/wsp-full:v1.0.0
ENV CLIENT_URL="https://your-business-server.com"
ENV CLIENT_ID="A_CLIENT_ID_FROM_YOLANDA_PLEASE_CONTACT_US"
# 16 digits or letters of the scale key, which must be consistent with the APP configuration.
ENV SCALE_KEY="1234567812345678"
ENV TZ="Asia/Shanghai"
```

> tips: The default timezone is UTC, you can change it by environment variable `TZ`. The log of Docker stores in log/access.log on a daily basis and keeps 7 days. The internal port of the container is 8080, and the external access port can be configured when Docker is started.

4. Build and run.

```shell
docker build . -t yolanda_wsp_full
docker run -p 8080:8080 yolanda_wsp_full
```

5. Map your application to a domain name.

Example: 0.0.0.0:8080 maps to https://your-docker-server.com 或 http://your-docker-server.com

## Endpoints between Docker and Scale 

#### Endpoint:

```text
GET https://your-docker-server.com/yolanda/wsp?code={encrypted_data}
```

## Endpoints between Docker and Server

### Check Users(Docker -> Server)

#### Endpoint

```text
POST https://your-business-server.com/wsp/list_users
```

#### Request Description

| Name | Type   | Required | Description | Extra                   |
| ---- | ------ | -------- | ----------- | ----------------------- |
| mac  | String | Y        | Mac Address | mock: 12:34:56:78:9A:BC |

#### Request Example

```json
{
    "mac": "12:34:56:78:9A:BC"
}
```

#### Response Description

| Name         | Type    | Required | Description                              | Extra            |
| ------------ | ------- | -------- | ---------------------------------------- | ---------------- |
| users        | array   | Y        | A List Of User Info                      | mock: []         |
| - user_index | integer | Y        | User's Index On The Scale                | mock: 1          |
| - gender     | integer | Y        | Gender                                   | mock: 1          |
| - height     | number  | Y        | Height(cm)                               | mock: 180.5      |
| - birthday   | string  | Y        | Birthday(YYYY-mm-dd)                     | mock: 1995-06-30 |
| - weight     | number  | Y        | User's Most Recently Measured Weight(kg) | mock: 78.6       |
| - goal_weight| number  | Y        | User's Goal Weight(kg)                   | mock: 70.0       |

#### Response Example

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

### Upload Measurement(Docker -> Server)

#### Endpoint

```text
POST https://your-business-server.com/wsp/create_measurements
```

#### Request Description

+ If only one record indicates that the user can be accurately identified.
+ If there are multiple records, it means that the user cannot be accurately identified, and multiple users claim ownership of the data.
+ The mechanism for confirming ownership is recommended: a pop-up page in the APP allows the user to choose whether the data is measured by himself or herself.

| Name            | Type    | Required | Description                     | Extra                        |
| --------------- | ------- | -------- | ------------------------------- | ---------------------------- |
| mac             | string  | Y        | Mac Address                     | mock: 12:34:56:78:9A:BC      |
| records         | array   | Y        | A List Of Measurement Records   | mock: []                     |
| - mac           | string  | Y        | Mac Address                     | mock: 12:34:56:78:9A:BC      |
| - model_id      | string  | Y        | Device Model (4 chars)          | mock: 0E2B                   |
| - user_index    | integer | Y        | User's Index On The Scale       | mock: 1                      |
| - timestamp     | integer | Y        | Measurement Timestamp (s)       | mock: 1582698882             |
| - weight        | number  | Y        | Weight (kg)                     | mock: 55.0                   |
| - heart_rate    | integer | Y        | Heart Rate (BPM)                | mock: 70                     |
| - hmac          | string  | Y        | Encrypted Signature             | mock: 183476B32E22B26989A... |
| - bmi           | number  | Y        | BMI                             | mock：20.1                   |
| - bodyfat       | number  | Y        | Body Fat Percentage (%)         | mock：14                     |
| - lbm           | number  | Y        | Lean Body Mass (kg)             | mock：50.1                   |
| - subfat        | number  | Y        | Subcutaneous Fat Percentage (%) | mock：12.7                   |
| - visfat        | number  | Y        | Visceral Fat Level              | mock：3.46                   |
| - water         | number  | Y        | Body Water Rate (%)             | mock：62.2                   |
| - bmr           | integer | Y        | BMR                             | mock：1451                   |
| - muscle        | number  | Y        | Skeletal Muscle Rate (%)        | mock：55.6                   |
| - sinew         | number  | Y        | Muscle Mass(kg)                 | mock：47.5                   |
| - bone          | number  | Y        | Bone Mass (kg)                  | mock：2.51                   |
| - protein       | number  | Y        | Protein Percentage (%)          | mock：19.5                   |
| - score         | number  | Y        | Health Assessment Score         | mock：90.2                   |
| - body_age      | integer | Y        | Body Age                        | mock：20                     |
| - body_shape    | integer | Y        | Body Shape                      | mock：4                      |
| - cardiac_index | number  | Y        | Cardiac Index                   | mock：0                      |

#### Request Example

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

#### Response Description

| Name       | Type | Required | Description    | Extra      |
| ---------- | ---- | -------- | -------------- | ---------- |
| is_success | bool | Y        | Success Or Not | mock: true |

#### Response Example

```json
{
   "is_success": true
}
```

### Eight Scale Upload Measurement(Docker -> Server)

#### Endpoint

```text
POST https://your-business-server.com/wsp/create_eight_measurements
```

#### Request Description

+ If only one record indicates that the user can be accurately identified.
+ If there are multiple records, it means that the user cannot be accurately identified, and multiple users claim ownership of the data.
+ The mechanism for confirming ownership is recommended: a pop-up page in the APP allows the user to choose whether the data is measured by himself or herself.

| Name                      | Type    | Required | Description                     | Extra                        |
| ------------------------- | ------- | -------- | ------------------------------- | ---------------------------- |
| mac                       | string  | Y        | Mac Address                     | mock: 12:34:56:78:9A:BC      |
| records                   | array   | Y        | A List Of Measurement Records   | mock: []                     |
| - mac                     | string  | Y        | Mac Address                     | mock: 12:34:56:78:9A:BC      |
| - model_id                | string  | Y        | Device Model (4 chars)          | mock: 0E2B                   |
| - user_index              | integer | Y        | User's Index On The Scale       | mock: 1                      |
| - timestamp               | integer | Y        | Measurement Timestamp (s)       | mock: 1582698882             |
| - weight                  | number  | Y        | Weight (kg)                     | mock: 55.0                   |
| - heart_rate              | integer | Y        | Heart Rate (BPM)                | mock: 70                     |
| - hmac                    | string  | Y        | Encrypted Signature             | mock: 183476B32E22B26989A... |
| - bmi                     | number  | Y        | BMI                             | mock：20.1                   |
| - bodyfat                 | number  | Y        | Body Fat Percentage (%)         | mock：14                     |
| - lbm                     | number  | Y        | Lean Body Mass (kg)             | mock：50.1                   |
| - subfat                  | number  | Y        | Subcutaneous Fat Percentage (%) | mock：12.7                   |
| - visfat                  | number  | Y        | Visceral Fat Level              | mock：3.46                   |
| - water                   | number  | Y        | Body Water Rate (%)             | mock：62.2                   |
| - bmr                     | integer | Y        | BMR                             | mock：1451                   |
| - muscle                  | number  | Y        | Skeletal Muscle Rate (%)        | mock：55.6                   |
| - sinew                   | number  | Y        | Muscle Mass(kg)                 | mock：47.5                   |
| - bone                    | number  | Y        | Bone Mass(kg)                   | mock：2.51                   |
| - protein                 | number  | Y        | Protein Percentage (%)          | mock：19.5                   |
| - score                   | number  | Y        | Health Assessment Score         | mock：90.2                   |
| - body_age                | integer | Y        | Body Age                        | mock：20                     |
| - body_shape              | integer | Y        | Body Shape                      | mock：4                      |
| - cardiac_index           | number  | Y        | Cardiac Index                   | mock：0                      |
| - right_arm_muscle_weight | number  | Y        | Muscle Mass Of Right Arm        | mock：2.83                   |
| - left_arm_muscle_weight  | number  | Y        | Muscle Mass Of Left Arm         | mock：2.77                   |
| - right_leg_muscle_weight | number  | Y        | Muscle Mass Of Right Leg        | mock：8.72                   |
| - left_leg_muscle_weight  | number  | Y        | Muscle Mass Of Left Leg         | mock：8.68                   |
| - trunk_muscle_weight     | number  | Y        | Muscle Mass Of Trunk            | mock：23.85                  |
| - right_arm_fat           | number  | Y        | Fat Percentage Of Right Arm     | mock：2.18                   |
| - left_arm_fat            | number  | Y        | Fat Percentage Of Left Arm      | mock：2.19                   |
| - right_leg_fat           | number  | Y        | Fat Percentage Of Right Leg     | mock：4.97                   |
| - left_leg_fat            | number  | Y        | Fat Percentage Of Left Leg      | mock：4.96                   |
| - trunk_fat               | number  | Y        | Fat Percentage Of Trunk         | mock：16.16                  |

#### Request Example

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

#### Response Description

| Name       | Type | Required | Description    | Extra      |
| ---------- | ---- | -------- | -------------- | ---------- |
| is_success | bool | Y        | Success Or Not | mock: true |

#### Response Example

```json
{
   "is_success": true
}
```

### APP interacts with Server

The interaction between `APP` and `Scale` needs to synchronize user's data, so `APP` needs to obtain the list of user information from `Server`, and save the data of the created user to `Server`.

`Server` needs to maintain the index, gender, birthday, height, and last weight in user's data.

When `APP` updates users, deletes users, and unbinds devices, `server` needs to maintain the consistency of user information.

like:

+ When `APP` updates a user, `Server` updates the user;
+ When `APP` deletes a user, `Server` deletes the user;
+ When `APP` unbinds a device, `Server` deletes the binding relationship between the user and the device .
