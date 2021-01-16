---
layout: post
title:  使用 Golang 判斷 MIME type 及讀取 CSV 檔案的內容
date:   2021-01-16
excerpt: "使用 gin-gonic/gin 和 github.com/gabriel-vasile/mimetype 套件"
category: Golang
tag:
- Golang
- CSV
- MIME type
- gin-gonic/gin
- github.com/gabriel-vasile/mimetype
comments: true
---

算是第一次正式用 Golang 寫程式，對於後端有很多不熟悉的地方，把這個功能的寫法紀錄一下，不確定是否有錯誤或不夠好的寫法，如果有更好的未來再更新。

## 使用的套件
* [gin-gonic/gin](<https://github.com/gin-gonic/gin#single-file> "Click")
* [github.com/gabriel-vasile/mimetype](<https://github.com/gabriel-vasile/mimetype> "Click")

## Code
go
```go
package main

import (
	"bufio"
	"bytes"
	"encoding/csv"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"github.com/gabriel-vasile/mimetype"
	"github.com/gin-gonic/gin"
)

// use "router.Use(CORS())"
func CORS() gin.HandlerFunc {
	return func(c *gin.Context) {
		c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
		c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")
		c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization, accept, origin, Cache-Control, X-Requested-With")
		c.Writer.Header().Set("Access-Control-Allow-Methods", "POST, OPTIONS, GET, PUT, DELETE")
		if c.Request.Method == "OPTIONS" {
			c.AbortWithStatus(204)
			return
		}
		c.Next()
	}
}

func main() {
	router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20 // 8 MiB
	router.Static("/", "./public")
	// 給前端使用的上傳檔案API：http://localhost:8080/csv/upload
	// port 的設定在最後一行
	router.Use(CORS()).POST("/csv/upload", func(c *gin.Context) {
		// get the file
		fFile, err := c.FormFile("file")
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"message": fmt.Sprintf("get file err: %s", err.Error()),
			})
			return
		}
		// open file
		file, err := fFile.Open()
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"message": err,
			})
			return
		}
		defer file.Close()
		// transfer file to data
		data, err := ioutil.ReadAll(bufio.NewReader(file))
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"message": fmt.Sprintf("err: %s", err),
			})
		}
		// detect if MIME type is "text/csv"
		mime := mimetype.Detect(data)
		if !mime.Is("text/csv") {
			c.JSON(http.StatusBadRequest, gin.H{
				"message": fmt.Sprintf("err: mime type is %s, text/csv is required", mime),
			})
			return
		}
		// read data
		reader := csv.NewReader(bytes.NewReader(data))
		line, err := reader.ReadAll()
		fmt.Println(line)
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"message": fmt.Sprintf("read err: %s", err.Error()),
			})
			return
		}

		// ... do with line

		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("File uploaded successfully"),
		})
	})
	router.Run(":8080") //port
}
```