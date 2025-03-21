{{ $section := "artifacts" }}
{{ range where .Site.Pages "Section" $section }}
- {{ .Title | markdownify }}: [{{ .Title }}]({{ .RelPermalink }})
{{ end }}