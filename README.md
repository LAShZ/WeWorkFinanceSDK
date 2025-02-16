# WeWorkFinanceSDK
企业微信会话存档SDK（基于企业微信C版官方SDK封装），暂时只支持在`linux`环境下使用当前SDK。

### 官方文档地址
https://open.work.weixin.qq.com/api/doc/90000/90135/91774

### 使用方式

#### 配置私有仓库

1、执行下面命令设置私有仓库地址到本地
> go env -w GOPRIVATE=github.com/NICEXAI

2、设置仓库访问凭证
> git config --global url."https://NICEXAI:a8e1cf7f5d15e8fb8c81fa7efc843af718db14a8@github.com".insteadOf "https://github.com"

#### 安装使用

1、安装 go module
> go get -u github.com/NICEXAI/WeWorkFinanceSDK

2、从 `github.com/NICEXAI/WeWorkFinanceSDK/lib` 文件夹下复制 `libWeWorkFinanceSdk_C.so` 动态库文件到系统动态链接库默认文件夹下，或者复制到任意文件夹并在当前文件夹下执行 `export LD_LIBRARY_PATH=$(pwd)`命令设置动态链接库检索地址

3、把 `module` 引入到项目中即可使用

### Example

```go
package main

import (
	"bytes"
	"fmt"
	"github.com/NICEXAI/WeWorkFinanceSDK"
	"io/ioutil"
	"os"
	"path"
)

func main() {
	corpID := "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
	corpSecret := "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
	rsaPrivateKey := `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`

	//初始化客户端
	client, err := WeWorkFinanceSDK.NewClient(corpID, corpSecret, rsaPrivateKey)
	if err != nil {
		fmt.Printf("SDK 初始化失败：%v \n", err)
		return
	}

	//同步消息
	chatDataList, err := client.GetChatData(0, 100, "", "", 3)
	if err != nil {
		fmt.Printf("消息同步失败：%v \n", err)
		return
	}

	for _, chatData := range chatDataList {
		//消息解密
		chatInfo, err := client.DecryptData(chatData.EncryptRandomKey, chatData.EncryptChatMsg)
		if err != nil {
			fmt.Printf("消息解密失败：%v \n", err)
			return
		}

		if chatInfo.Type == "image" {
			image := chatInfo.GetImageMessage()
			sdkfileid := image.Image.SdkFileId

			isFinish := false
			buffer := bytes.Buffer{}
			for !isFinish {
				//获取媒体数据
				mediaData, err := client.GetMediaData("", sdkfileid, "", "", 5)
				if err != nil {
					fmt.Printf("媒体数据拉取失败：%v \n", err)
					return
				}
				buffer.Write(mediaData.Data)
				if mediaData.IsFinish {
					isFinish = mediaData.IsFinish
				}
			}
			filePath, _ := os.Getwd()
			filePath = path.Join(filePath, "test.png")
			err := ioutil.WriteFile(filePath, buffer.Bytes(), 0666)
			if err != nil {
				fmt.Printf("文件存储失败：%v \n", err)
				return
			}
			break
		}
	}
}



```
