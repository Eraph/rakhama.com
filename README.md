# Rakhama.com

## [Hugo](https://gohugo.io/)
### Create new post
``` bash
hugo new posts/<post-name>/index.md
```

### Create new page
``` bash
hugo new pages/<page-name>/index.md
```

### Compile
``` bash
hugo --cleanDestinationDir
```

### Watch
``` bash
hugo -w --cleanDestinationDir
```

### Compile with drafts
``` bash
hugo -D --cleanDestinationDir
```

### Watch with drafts
``` bash
hugo -Dw --cleanDestinationDir
```

### Debug
{{ range .Pages }}
    {{ printf "%#v" . }}
{{ end }}
