# tbot - Telegram Bot Server [![Build Status](https://travis-ci.org/bohdanlisovskyi/telegrambot/tbot.svg?branch=master)](https://travis-ci.org/bohdanlisovskyi/telegrambot/tbot) [![Go Report Card](https://goreportcard.com/badge/github.com/bohdanlisovskyi/telegrambot/tbot)](https://goreportcard.com/report/github.com/bohdanlisovskyi/telegrambot/tbot) [![codecov](https://codecov.io/gh/bohdanlisovskyi/telegrambot/tbot/branch/master/graph/badge.svg)](https://codecov.io/gh/bohdanlisovskyi/telegrambot/tbot)
[![GoDoc](https://godoc.org/github.com/bohdanlisovskyi/telegrambot/tbot?status.svg)](https://godoc.org/github.com/bohdanlisovskyi/telegrambot/tbot)

**tbot** is a Telegram bot server.

## Installation

```bash
go get -u github.com/bohdanlisovskyi/telegrambot/tbot
```

## Support

Join [telegram group](https://t.me/tbotgo) to get support.

## Usage

It feels like net/http Server, so it's easy to use:

[embedmd]:# (examples/simple/main.go)
```go
package main

import (
	"log"
	"os"
	"time"

	"github.com/bohdanlisovskyi/telegrambot/tbot"
)

func main() {
	token := os.Getenv("TELEGRAM_TOKEN")
	// Create new telegram bot server using token
	bot, err := tbot.NewServer(token)
	if err != nil {
		log.Fatal(err)
	}

	// Use whitelist for Auth middleware, allow to interact only with user1 and user2
	whitelist := []string{"bohdanlisovskyi/telegrambot", "user2"}
	bot.AddMiddleware(tbot.NewAuth(whitelist))

	// Yo handler works without slash, simple text response
	bot.Handle("yo", "YO!")

	// Handle with HiHandler function
	bot.HandleFunc("/hi", HiHandler)
	// Handler can accept varialbes
	bot.HandleFunc("/say {text}", SayHandler)
	// Bot can send stickers, photos, music
	bot.HandleFunc("/sticker", StickerHandler)
	bot.HandleFunc("/photo", PhotoHandler)
	bot.HandleFunc("/keyboard", KeyboardHandler)

	// Use file handler to handle user uploads
	bot.HandleFile(FileHandler)

	// Set default handler if you want to process unmatched input
	bot.HandleDefault(EchoHandler)

	// Start listening for messages
	err = bot.ListenAndServe()
	log.Fatal(err)
}

func HiHandler(message *tbot.Message) {
	// Handler can reply with several messages
	message.Replyf("Hello, %s!", message.From)
	time.Sleep(1 * time.Second)
	message.Reply("What's up?")
}

func SayHandler(message *tbot.Message) {
	// Message contain it's varialbes from curly brackets
	message.Reply(message.Vars["text"])
}

func EchoHandler(message *tbot.Message) {
	message.Reply(message.Text())
}

func StickerHandler(message *tbot.Message) {
	message.ReplySticker("sticker.png")
}

func PhotoHandler(message *tbot.Message) {
	message.ReplyPhoto("photo.jpg", "it's me")
}

func KeyboardHandler(message *tbot.Message) {
	buttons := [][]string{
		{"Some", "Test", "Buttons"},
		{"Another", "Row"},
	}
	message.ReplyKeyboard("Buttons example", buttons)
}

func FileHandler(message *tbot.Message) {
	err := message.Download("./uploads")
	if err != nil {
		message.Replyf("Error handling file: %q", err)
		return
	}
	message.Reply("Thanks for uploading!")
}
```

See full documentation here: https://godoc.org/github.com/bohdanlisovskyi/telegrambot/tbot

Test coverage: https://gocover.io/github.com/bohdanlisovskyi/telegrambot/tbot
