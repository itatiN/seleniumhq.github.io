---
title: "Funcionalidade específica do Safari"
linkTitle: "Safari"
weight: 10
description: >-
    Estas capacidades e características são específicas ao navegador Apple Safari.
aliases: [
"/pt-br/documentation/capabilities/safari"
]
---

Ao invés dos drivers para Chromium e Firefox, o safaridriver faz parte to sistema Operativo.
Para activar a automação no Safari, execute o seguinte comando no terminal:

```shell
safaridriver --enable
```

## Opções

Capacidades comuns a todos os navegadores estão descritas na [página Opções]({{< ref "../drivers/options.md" >}}).

Capacidades únicas ao Safari podem ser encontradas na página da Apple [WebDriver para Safari](https://developer.apple.com/documentation/webkit/about_webdriver_for_safari#2957227)

Este é um exemplo de como iniciar uma sessão Safari com um conjunto de opções básicas::

{{< tabpane code=false langEqualsHeader=true >}}
{{< tab header="Java" >}}
{{< gh-codeblock path="/examples/java/src/test/java/dev/selenium/browsers/SafariTest.java#21-L22" >}}
{{< /tab >}}
{{< tab header="Python" >}}
{{< gh-codeblock path="/examples/python/tests/browsers/test_internet_explorer.py#L10-L11" >}}
{{< /tab >}}
{{< tab header="CSharp" >}}
{{< gh-codeblock path="/examples/dotnet/SeleniumDocs/Browsers/SafariTest.cs#L14-L15" >}}
{{< /tab >}}
{{< tab header="Ruby" >}}
{{< gh-codeblock path="/examples/ruby/spec/browsers/internet_explorer_spec.rb#L8-L9" >}}
{{< /tab >}}
{{< tab header="JavaScript" >}}
  let options = new safari.Options();
  let driver = await new Builder()
    .forBrowser('safari')
    .setSafariOptions(options)
    .build();
{{< /tab >}}
{{< tab header="Kotlin" >}}
  val options = SafariOptions()
  val driver = SafariDriver(options)
{{< /tab >}}
{{< /tabpane >}}

### Mobile (celular)
Se pretende automatizar Safari em iOS, deve olhar para o [Projecto Appium](//appium.io/).