---
title: GoのCSRF対策
date: 2025-11-12
modified: 2025-12-26
tags:
  - go
  - tips
---
- https://future-architect.github.io/articles/20250804a/
- https://blog.jxck.io/entries/2024-04-26/csrf.html
- ginでの実装demo(1.25からの機能なので無理くり...)

```go
import (
	"log/slog"
	"net/http"

	"github.com/gin-gonic/gin"
)

func CSRFMiddleware(logger *slog.Logger) gin.HandlerFunc {
	// CSRF保護の設定
	csrfProtection := http.NewCrossOriginProtection()

	// 信頼オリジンの設定
	err := csrfProtection.AddTrustedOrigin("http://localhost:3000")
	if err != nil {
		logger.Error("not add origin", "error", err)
	}

	// カスタムdenyハンドラの設定
	denyHandler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		logger.Error("Origin Blocked", "origin", r.Header.Get("Origin"), "path", r.URL.Path)
		http.Error(w, "Forbidden", http.StatusForbidden)
	})
	csrfProtection.SetDenyHandler(denyHandler)

	return func(c *gin.Context) {
		csrfProtection.Handler(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			c.Next()
		})).ServeHTTP(c.Writer, c.Request)
	}
}
```
