---
author: ["Me"]
title: 'TOTP in Go'
date: 2024-03-24T15:18:01+08:00
categories: ["Programming", "Security"]
tags: ["Go", "TOTP", "Authentication"]
draft: false
---

一个很小的 TOTP 的实现 代码写的非常简洁易懂 代码注释已经说明了每一步的实现

```go
package main

import (
        "crypto/hmac"
        "crypto/sha1"
        "encoding/base32"
        "encoding/binary"
        "fmt"
        "strings"
        "time"
)

func generateTOTP(secretKey string, timestamp int64) uint32 {
        // The base32 encoded secret key string is decoded to a byte slice
        base32Decoder := base32.StdEncoding.WithPadding(base32.NoPadding)
        secretKey = strings.ToUpper(strings.TrimSpace(secretKey)) // preprocess
        secretBytes, _ := base32Decoder.DecodeString(secretKey)   // decode

        // The truncated timestamp / 30 is converted to an 8-byte big-endian
        // unsigned integer slice
        timeBytes := make([]byte, 8)
        binary.BigEndian.PutUint64(timeBytes, uint64(timestamp)/30)

        // The timestamp bytes are concatenated with the decoded secret key
        // bytes. Then a 20-byte SHA-1 hash is calculated from the byte slice
        hash := hmac.New(sha1.New, secretBytes)
        hash.Write(timeBytes) // Concat the timestamp byte slice
        h := hash.Sum(nil)    // Calculate 20-byte SHA-1 digest

        // AND the SHA-1 with 0x0F (15) to get a single-digit offset
        offset := h[len(h)-1] & 0x0F

        // Truncate the SHA-1 by the offset and convert it into a 32-bit
        // unsigned int. AND the 32-bit int with 0x7FFFFFFF (2147483647)
        // to get a 31-bit unsigned int.
        truncatedHash := binary.BigEndian.Uint32(h[offset:]) & 0x7FFFFFFF

        // Take modulo 1_000_000 to get a 6-digit code
        return truncatedHash % 1_000_000
}

func main() {
        // Collect it from a TOTP server like GitHub 2FA panel
        secretKey := "ddddd" // This is a fake one!

        now := time.Now().Unix()
        totpCode := generateTOTP(secretKey, now)

        fmt.Printf("Current TOTP code: %06d\n", totpCode)
}
```


Ported from: https://rednafi.com/go/totp_client/

至于当前有效时间，我们可以简单做一下增强。按照一般的间隔 30s 对当前的 unixtimestamp 做一下求余即可得到。

```go
// ...
const (
	INTERVAL = 30
)
// ...
func main()
	// Collect it from a TOTP server like GitHub 2FA panel
	secretKey := "dddd" // This is a fake one!

	now := time.Now().Unix()
	totpCode := generateTOTP(secretKey, now)

	remainingSecs := INTERVAL - now%INTERVAL

	fmt.Printf("Current TOTP code: %06d Remaining: %02ds\n", totpCode, remainingSecs)
```
