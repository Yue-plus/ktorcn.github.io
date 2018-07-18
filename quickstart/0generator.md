---
title: 生成器
caption: 生成一个 Ktor 项目
category: quickstart
permalink: /quickstart/generator.html
redirect_from:
  - /quickstart/quickstart/generator.html
---

<!--<https://ktor.io/start>-->

**注：除了在线生成，还可以使用 [Ktor IntelliJ 插件](/quickstart/quickstart/intellij-idea/plugin.html)。**

<div id="generator_id"></div>

<script type="text/javascript">
window.addEventListener('popstate', function(event) {
    const iframe = document.getElementById('iframe_generator');
    if (iframe) {
        iframe.contentWindow.postMessage({type: "updateHash", value: window.location.hash}, "*")
    }
});
window.addEventListener('message', function(event) {
    if (event.data && event.data.type === "updateHash") {
        history.pushState({}, "", window.location.pathname + "#" + event.data.value.replace(/^#/, ''));
    }
});
document.getElementById('generator_id').innerHTML = '<iframe id="iframe_generator" src="{{ site.ktor_init_tools_url }}' + location.hash.replace(/"/g, '\\"') + '" style="border:1px solid #343a40;width:100%;height:500px;"></iframe>';
</script>
