```
package main

import (
	"log"
	"os"
)

var logger *log.Logger

func main() {
	file, err := os.OpenFile("test.log", os.O_APPEND|os.O_CREATE, 666)
	if err != nil {
		log.Fatalln("fail to create test.log file!")
	}
	defer file.Close()
	logger = log.New(file, "", log.LstdFlags|log.Lshortfile) // 日志文件格式:log包含时间及文件行数

	log.Print("我就是一条日志")
	log.Printf("%s,","谁说我是日志了，我是错误")
	log.Panic("哈哈，我好痛")
	log.Println("输出日志到命令行终端")
    log.Panicln("在命令行终端输出panic，并中断程序执行")
    log.Fatal("在命令行终端输出日志并执行os.exit(1)")

    logger.SetFlags(log.LstdFlags | log.Lshortfile) // 设置日志格式
	logger.Println("将日志写入文件")	
	logger.Panicln("在日志文件中写入panic，并中断程序执行")	
	logger.Fatal("在日志文件中写入日志并执行os.exit(1)")
}
```