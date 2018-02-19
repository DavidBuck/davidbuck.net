+++
title = "Configure Hugo to ouput a single RSS feed for blog posts only"
date = 2018-02-01T21:16:37+11:00
+++

By default, [Hugo](//gohugo.io) generates RSS files for every [section](//gohugo.io/content-management/sections/) your site has. These are generated as `index.xml` files in each section directory (eg `post/index.xml`, `project/index.xml`) and list all pages for that section. An `index.xml` file is also rendered at the root of your project and includes items from all secions of your site.

You can configure Hugo to generate a single RSS file listing only posts from a single section (e.g blog posts). A custom RSS file name can also be set (e.g. `feed.xml`).

## 1. Ouput a single RSS file

To configure Hugo to only output a sinlge RSS file to the root of your project, add the following to your `config.toml`:

{{< highlight toml >}}
[outputs]
home = [ "RSS", "HTML"]
{{< /highlight >}}

source: [Hugo community](https://discourse.gohugo.io/t/how-can-i-change-the-rss-url/118/17)

## 2. Set the RSS file to feed.xml

To set a custom name for the RSS file, add the following to your `config.toml`:

{{< highlight toml >}}
[outputFormats]
[outputFormats.RSS]
mediatype = "application/rss"
baseName = "feed"
{{< /highlight >}}

You can change the `baseName` from *feed* to your preferred file name.

source: [Hugo community](https://discourse.gohugo.io/t/how-can-i-change-the-rss-url/118/17)

## 3. Set the RSS file at the project root to display only blog posts

Hugo uses a [default RSS template](//gohugo.io/templates/rss/#the-embedded-rss-xml) to generate the RSS files. The template code is shown below:

{{< highlight xml "linenos=inline,hl_lines=15" >}}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ if eq  .Title  .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} on {{ end }}{{ .Site.Title }}{{ end }}</title>
    <link>{{ .Permalink }}</link>
    <description>Recent content {{ if ne  .Title  .Site.Title }}{{ with .Title }}in {{.}} {{ end }}{{ end }}on {{ .Site.Title }}</description>
    <generator>Hugo -- gohugo.io</generator>{{ with .Site.LanguageCode }}
    <language>{{.}}</language>{{end}}{{ with .Site.Author.email }}
    <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
    <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
    <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
    <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
    {{ with .OutputFormats.Get "RSS" }}
        {{ printf "<atom:link href=%q rel=\"self\" type=%q />" .Permalink .MediaType | safeHTML }}
    {{ end }}
    {{ range .Data.Pages }}
    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
      {{ with .Site.Author.email }}<author>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</author>{{end}}
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Summary | html }}</description>
    </item>
    {{ end }}
  </channel>
</rss>
{{< /highlight >}}

The default template uses `{{ range .Data.Pages }}` to loop over the pages in a section or, for the index.xml at the root, all pages in every section.

To include only your blog posts in the feed copy the default RSS template code to `layouts\rss.xml` in your Hugo project. Then change line 15 in the new `rss.xml` file, from:

`{{ range .Data.Pages }}`

to the following, which limits the range to sections of type post:

`{{ range where .Site.RegularPages "Section" "post" }}`

You can change `"post"` to the appropriate section in your site.

## 4. Adding a link to feed.xml

Now you have a single RSS file, you can link to it in the html `<head>` of your template files. To include a link to `feed.xml` at your site root add the following to your `header.html` partial or relevant template:

{{< highlight html >}}
<link rel="alternate" type="application/rss+xml" href='{{ "feed.xml" | absURL }}' />
{{< /highlight >}}
