# 使用go实现猜字游戏和翻译工具 | 青训营笔记

这是我参与「第三届青训营 -后端场」笔记创作活动的的第1篇笔记，青训营day1主要介绍了go的基础语法，同时提供了一些项目作为练习，猜字游戏侧重于go的基础语法以及输入输出相关package的用法，翻译工具更侧重于如何发起HTTP请求及响应。

## 猜字游戏

首先描述一下功能：运行后内部产生一个random，然后提示用户输入一个随机数，如果用户输入不是数字则提示重新输入，如果是数字则对比大小后给出提示。

```go
func main() {
	maxNum := 100
	//随机数种子
	rand.Seed(time.Now().UnixNano())
	secretNumber := rand.Intn(maxNum)

    //提示
	fmt.Println("Please input your guess")
	for {
		var guess int

        //和c的scanf一样，传入addr
		_, err := fmt.Scanf("%d\r\n", &guess)

        //无法parseInt
		if err != nil {
			fmt.Println("Invalid input. Please enter an integer value")
			continue
		}
		fmt.Println("You guess is", guess)
		if guess > secretNumber {
			fmt.Println("Your guess is bigger than the secret number. Please try again")
		} else if guess < secretNumber {
			fmt.Println("Your guess is smaller than the secret number. Please try again")
		} else {
			fmt.Println("Correct, you Legend!")
			break
		}
	}
}
复制代码
```

**tips:fmt.Scanf对于win用户要注意换行，windows输入回车后大概率会包含\r\n，如果不处理则无法转换成int。mac OS和linux还没有测试**

## 翻译工具

这个工具逻辑上比较简单，大概流程是：接受输入单词 -> 构建client和request -> 将data作为payload以post发送 -> 将response转化为struct

```go
//main
func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, `usage: simpleDict WORD
example: simpleDict hello
		`)
		os.Exit(1)
	}
	word := os.Args[1]
	for i := 1; i < runtime.NumCPU(); i++{
		query(word)
	}
}

//query 核心请求函数
func query(word string) {
	client := &http.Client{}

    //request data
	request := DictRequest{TransType: "en2zh", Source: word}
	buf, err := json.Marshal(request)
	if err != nil {
		log.Fatal(err)
	}
	var data = bytes.NewReader(buf)

    //HTTP请求
	req, err := http.NewRequest("POST", "https://api.interpreter.caiyunai.com/v1/dict", data)
	if err != nil {
		log.Fatal(err)
	}

    // Set Header

	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}

    //defer关闭
	defer resp.Body.Close()
	bodyText, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

    //bad request
	if resp.StatusCode != 200 {
		log.Fatal("bad StatusCode:", resp.StatusCode, "body", string(bodyText))
	}
	var dictResponse DictResponse
	err = json.Unmarshal(bodyText, &dictResponse)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(word, "UK:", dictResponse.Dictionary.Prons.En, "US:", dictResponse.Dictionary.Prons.EnUs)
	for _, item := range dictResponse.Dictionary.Explanations {
		fmt.Println(item)
	}
}
复制代码
```

这里面有几个要注意的点：

- 请求和响应的数据结构依赖于struct，定义好json flag后方便转换
- 构建req是指定method url和data
- 记得判断请求状态，是success还是error