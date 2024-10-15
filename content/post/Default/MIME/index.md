---
title: MIME类型
description: MIME类型
date: 2023-05-26
slug: mime_type
image: 
categories:
    - Default
tags:
    - MIME
---

# MIME类型
## 参考文档
- [IANA官方MIME类型大全](https://www.iana.org/assignments/media-types/media-types.xhtml)
- [IBM Integration Bus v10.1](https://www.ibm.com/docs/zh/integration-bus/10.1?topic=information-additional-mime-domain)
- [MDN Web Docs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_Types)

## MIME介绍
MIME（Multipurpose Internet Mail Extensions，邮件扩展类型）是一种标准的、用来表示文档、文件、字节流等数据的格式和性质的一种流媒体类型。

浏览器通常采用MIME类型而不是文件扩展名来确定如何处理URL和文件内容。所以在响应头需要添加标准的mimetype，否则浏览器将不能以正常的方式来处理解析文件内容。
## MIME语法
通用结构：
```
type/subtype
```
MIME的组成结构；由类型与子类型两个字符串中间用`/`分隔而组成。不允许空格存在。type表示可以被分多个子类的独立类别。subtype表示细分后的每个类型。

MIME类型对大小写不敏感，但是传统写法都是小写。

## mimetype与Content-Type
在打开网站的调试模式的时候，我们可以看到，在请求的响应标头往往有一个Content-Type的属性，Content-Type的第一个属性往往都是mimetype的类型，通过`;`来分割。

从MIME的全称和其历史，MIME最开始的提出是为了解决邮件编码而扩展的一种类型。邮件协议在最开始提出的时候，只能传输一定长度的ASCII码文件，随着计算机网络的发展，单纯的ASCII码显然不能满足世界的多样化需求。所以此时MIME就诞生了，MIME作为电子邮件协议SMTP的Extensions扩展，它并没有取代或改变SMTP协议，而仅仅是对SMTP协议的一种扩展。通过MIME协议，使得邮件传输过程中不会被邮件协议而改变。

万维网和HTTP协议的诞生要晚于邮件协议的诞生，在HTTP协议的发展中，广泛吸收了其他协议的优点，如MIME。在HTTP协议中，通过`Content-Type`标头来确定HTTP传输的内容，从其功能来看，可以确定是借鉴了MIME。且在`Content-Type`标头中，通常其第一个值为MIME规范中所支持的mimetype，其他属性通过过`;`来分割。所以可以说`Content-Type`是`mimetype`的超集，在`Content-Type`中包含了对`mimetype`的定义和使用。

## MIME类型大全
### 常用MIME类型
| 文件类型 | MIME类型 | 描述 |
| --- | --- | --- |
| .txt | text/plain | 通常用于典型的邮件或新闻消息。 text/richtext 也是常用值 |
| .csv | text/csv |
| .css | text/css | 
| .xml | text/xml | xml对用户可读（不渲染） |
| .js | text/javascript |
| .jpg \| .jpeg | image/jpeg | 用于图像。 image/jpeg 和 image/gif 是使用的公共图像格式 | 
| .gif | image/gif |
| .xml | application/xml | xml对用户不可读（渲染） |
| .bin | application/octet-stream | 当消息是未知类型和包含任何字节数据时使用 |
| .pdf | application/pdf | 
| .json | application/json | 
| .epub | application/epub+zip |
| .doc | application/msword |
| .docx | application/vnd.openxmlformats-officedocument.wordprocessingml.document |
| .ppt | application/vnd.ms-powerpoint |
| .pptx | application/vnd.openxmlformats-officedocument.presentationml.presentation |
| .jar | application/java-archive |
| .rar | application/x-rar-compressed |
| .tar | application/x-tar |
| .zip | application/zip |
| .7z | application/x-7z-compressed |


### application类型
| 文件类型 | MIME类型 |
| --- | --- |
| .xlsx | application/vnd.openxmlformats-officedocument.spreadsheetml.sheet |
| .xltx | application/vnd.openxmlformats-officedocument.spreadsheetml.template |
| .potx | application/vnd.openxmlformats-officedocument.presentationml.template |
| .ppsx | application/vnd.openxmlformats-officedocument.presentationml.slideshow |
| .pptx | application/vnd.openxmlformats-officedocument.presentationml.presentation |
| .sldx | application/vnd.openxmlformats-officedocument.presentationml.slide |
| .docx | application/vnd.openxmlformats-officedocument.wordprocessingml.document |
| .dotx | application/vnd.openxmlformats-officedocument.wordprocessingml.template |
| .xlam | application/vnd.ms-excel.addin.macroEnabled.12 |
| .xlsb | application/vnd.ms-excel.sheet.binary.macroEnabled.12 |
| .epub | application/epub+zip |
| .apk | application/vnd.android.package-archive |
| .hqx | application/mac-binhex40 |
| .cpt | application/mac-compactpro |
| .doc | application/msword |
| .pdf | application/pdf |
| .mif | application/vnd.mif |
| .xls | application/vnd.ms-excel |
| .ppt | application/vnd.ms-powerpoint |
| .odc | application/vnd.oasis.opendocument.chart |
| .odb | application/vnd.oasis.opendocument.database |
| .odf | application/vnd.oasis.opendocument.formula |
| .odg | application/vnd.oasis.opendocument.graphics |
| .otg | application/vnd.oasis.opendocument.graphics-template |
| .odi | application/vnd.oasis.opendocument.image |
| .odp | application/vnd.oasis.opendocument.presentation |
| .otp | application/vnd.oasis.opendocument.presentation-template |
| .ods | application/vnd.oasis.opendocument.spreadsheet |
| .ots | application/vnd.oasis.opendocument.spreadsheet-template |
| .odt | application/vnd.oasis.opendocument.text |
| .odm | application/vnd.oasis.opendocument.text-master |
| .ott | application/vnd.oasis.opendocument.text-template |
| .oth | application/vnd.oasis.opendocument.text-web |
| .sxw | application/vnd.sun.xml.writer |
| .stw | application/vnd.sun.xml.writer.template |
| .sxc | application/vnd.sun.xml.calc |
| .stc | application/vnd.sun.xml.calc.template |
| .sxd | application/vnd.sun.xml.draw |
| .std | application/vnd.sun.xml.draw.template |
| .sxi | application/vnd.sun.xml.impress |
| .sti | application/vnd.sun.xml.impress.template |
| .sxg | application/vnd.sun.xml.writer.global |
| .sxm | application/vnd.sun.xml.math |
| .sis | application/vnd.symbian.install |
| .wbxml | application/vnd.wap.wbxml |
| .wmlc | application/vnd.wap.wmlc |
| .wmlsc | application/vnd.wap.wmlscriptc |
| .bcpio | application/x-bcpio |
| .torrent | application/x-bittorrent |
| .bz2 | application/x-bzip2 |
| .vcd | application/x-cdlink |
| .pgn | application/x-chess-pgn |
| .cpio | application/x-cpio |
| .csh | application/x-csh |
| .dvi | application/x-dvi |
| .spl | application/x-futuresplash |
| .gtar | application/x-gtar |
| .hdf | application/x-hdf |
| .jar | application/java-archive |
| .jnlp | application/x-java-jnlp-file |
| .js | application/javascript |
| .json | application/json |
| .ksp | application/x-kspread |
| .chrt | application/x-kchart |
| .kil | application/x-killustrator |
| .latex | application/x-latex |
| .rpm | application/x-rpm |
| .sh | application/x-sh |
| .shar | application/x-shar |
| .swf | application/x-shockwave-flash |
| .sit | application/x-stuffit |
| .sv4cpio | application/x-sv4cpio |
| .sv4crc | application/x-sv4crc |
| .rar | application/x-rar-compressed |
| .tar | application/x-tar |
| .7z | application/x-7z-compressed |
| .tcl | application/x-tcl |
| .tex | application/x-tex |
| .man | application/x-troff-man |
| .me | application/x-troff-me |
| .ms | application/x-troff-ms |
| .ustar | application/x-ustar |
| .src | application/x-wais-source |
| .zip | application/zip |
| .ai | application/postscript |
| .atom | application/atom+xml |
| .bin | application/octet-stream |
| .cdf | application/x-netcdf |
| .class | application/octet-stream |
| .dcr | application/x-director |
| .dir | application/x-director |
| .dll | application/octet-stream |
| .dmg | application/octet-stream |
| .dms | application/octet-stream |
| .dtd | application/xml-dtd |
| .dxr | application/x-director |
| .eps | application/postscript |
| .exe | application/octet-stream |
| .ez | application/andrew-inset |
| .gram | application/srgs |
| .grxml | application/srgs+xml |
| .gz | application/x-gzip |
| .lha | application/octet-stream |
| .lzh | application/octet-stream |
| .mathml | application/mathml+xml |
| .nc | application/x-netcdf |
| .oda | application/oda |
| .ps | application/postscript |
| .rdf | application/rdf+xml |
| .rm | application/vnd.rn-realmedia |
| .roff | application/x-troff |
| .skd | application/x-koan |
| .skm | application/x-koan |
| .skp | application/x-koan |
| .skt | application/x-koan |
| .smi | application/smil |
| .smil | application/smil |
| .so | application/octet-stream |
| .t | application/x-troff |
| .texi | application/x-texinfo |
| .texinfo | application/x-texinfo |
| .tr | application/x-troff |
| .vxml | application/voicexml+xml |
| .xht | application/xhtml+xml |
| .xhtml | application/xhtml+xml |
| .xml | application/xml |
| .xsl | application/xml |
| .xslt | application/xslt+xml |
| .xul | application/vnd.mozilla.xul+xml |
### text类型
| 文件类型 | MIME类型 |
| --- | --- |
| .txt | text/plain |
| .asc | text/plain |
| .csv | text/csv |
| .htm | text/html |
| .html | text/html |
| .xml | text/xml |
| .css | text/css |
| .rtf | text/rtf |
| .rtx | text/richtext |
| .ics | text/calendar |
| .ifb | text/calendar |
| .tsv | text/tab-separated-values |
| .jad | text/vnd.sun.j2me.app-descriptor |
| .wml | text/vnd.wap.wml |
| .wmls | text/vnd.wap.wmlscript |
| .etx | text/x-setext |
| .sgm | text/sgml |
| .sgml | text/sgml |
### image类型
| 文件类型 | MIME类型 |
| --- | --- |
|.jpe | image/jpeg |
| .jpg | image/jpeg |
| .jpeg | image/jpeg |
| .png | image/png |
| .gif | image/gif |
| .bmp | image/bmp |
| .ico | image/x-icon |
| .svg | image/svg+xml |
| .webp | image/webp |
| .wbmp | image/vnd.wap.wbmp |
| .tif | image/tiff |
| .tiff | image/tiff |
| .ief | image/ief |
| .rgb | image/x-rgb |
| .xbm | image/x-xbitmap |
| .xpm | image/x-xpixmap |
| .xwd | image/x-xwindowdump |
| .ras | image/x-cmu-raster |
| .pnm | image/x-portable-anymap |
| .pbm | image/x-portable-bitmap |
| .pgm | image/x-portable-graymap |
| .ppm | image/x-portable-pixmap |
| .cgm | image/cgm |
| .djv | image/vnd.djvu |
| .djvu | image/vnd.djvu |
| .jp2 | image/jp2 |
| .mac | image/x-macpaint |
| .pct | image/pict |
| .pic | image/pict |
| .pict | image/pict |
| .pnt | image/x-macpaint |
| .pntg | image/x-macpaint |
| .qti | image/x-quicktime |
| .qtif | image/x-quicktime |
### video类型
| 文件类型 | MIME类型 |
| --- | --- |
| .mp4 | video/mp4 |
| .mpeg | video/mpeg |
| .mov | video/quicktime |
| .qt | video/quicktime |
| .avi | video/x-msvideo |
| .flv | video/x-flv |
| .webm | video/webm |
| .mxu | video/vnd.mpegurl |
| .wm | video/x-ms-wm |
| .wmv | video/x-ms-wmv |
| .wmx | video/x-ms-wmx |
| .wvx | video/x-ms-wvx |
| .movie | video/x-sgi-movie |
| .3gp | video/3gpp |
| .dif | video/x-dv |
| .dv | video/x-dv |
| .m4u | video/vnd.mpegurl |
| .m4v | video/x-m4v |
| .mpe | video/mpeg |
| .mpg | video/mpeg |
| .ogv | video/ogv |
### audio类型
| 文件类型 | MIME类型 |
| --- | --- |
| .mp2 | audio/mpeg |
| .mp3 | audio/mpeg |
| .aif | audio/x-aiff |
| .aifc | audio/x-aiff |
| .aiff | audio/x-aiff |
| .ogg | audio/ogg |
| .m3u | audio/x-mpegurl |
| .ra | audio/x-pn-realaudio |
| .wav | audio/x-wav |
| .wma | audio/x-ms-wma |
| .wax | audio/x-ms-wax |
| .au | audio/basic |
| .kar | audio/midi |
| .m4a | audio/mp4a-latm |
| .m4p | audio/mp4a-latm |
| .mid | audio/midi |
| .midi | audio/midi |
| .mpga | audio/mpeg |
| .ram | audio/x-pn-realaudio |
| .snd | audio/basic |
### model类型
| 文件类型 | MIME类型 |
| --- | --- |
| .iges | model/iges |
| .igs | model/iges |
| .mesh | model/mesh |
| .msh | model/mesh |
| .silo | model/mesh |
| .vrml | model/vrml |
| .wrl | model/vrml |
### 其他类型
| 文件类型 | MIME类型 |
| --- | --- |
| .pdb | chemical/x-pdb |
| .xyz | chemical/x-xyz |
| .ice | x-conference/x-cooltalk |
### multipart复合类型
| MIME类型 | 描述 |
| --- | --- |
| multipart/form-data | 可用于联系 HTML Forms 和 POST 方法 |
| multipart/byteranges | 使用状态码206 Partial Content来发送整个文件的子集 |
| multipart/related | 用于消息中多个相关部分。 具体地讲，与 SwA（具有附件的 SOAP）一起使用 |
| multipart/signed | 用于消息中多个相关部分（包括签名）。 具体的讲，与 S/MIME 一起使用 |
| multipart/mixed | 用于消息中多个独立部分。|