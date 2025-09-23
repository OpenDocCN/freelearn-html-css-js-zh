# 第四章：内容处理

Bootstrap 4 中的内容组件是为最常用的 HTML 元素保留的，例如图片、表格、排版等。在本章中，我将教您如何使用构建任何类型网站或网络应用程序所需的全部 HTML 构建块。我们将从每个组件的快速概述开始，然后将其构建到我们的博客项目中。Bootstrap 4 附带了一个全新的 CSS 重置，称为 Reboot。它建立在 `Normalize.css` 之上，使您的网站在开箱即用的情况下看起来更好。在我们深入之前，让我们回顾一下在 Bootstrap 4 中处理内容组件时的 Reboot 基础知识。

# Reboot 默认值和基础知识

让我们从回顾 Bootstrap 中使用内容组件时 Reboot 的基础知识开始这一章。Bootstrap 4 中内容组件的主要变化之一是从 `em` 单位切换到 `rem` 单位。`rem` 是 `root em` 的缩写，与常规的 `em` 单位略有不同。`em` 是相对于其包含的父元素字体大小的相对单位。这可能导致代码中出现累积问题，当您有一组高度嵌套的选择器时，这些问题可能难以处理。`rem` 单位不是相对于其父元素，而是相对于根或 HTML 元素。这使得确定将在屏幕上显示的实际文本或其他内容组件的大小变得容易得多。

`box-sizing` 属性在所有元素上全局设置为 `border-box`。这很好，因为它确保元素的声明宽度不会因为额外的边距或填充而超过其大小。

Bootstrap 4 的基本 `font-size` 为 `16px`，并在 `html` 元素上声明。在 `body` 元素上，`font-size` 被设置为 `1rem`，以便在使用媒体查询时轻松进行响应式类型缩放。

在 `body` 标签上还设置了全局的 `font-family` 和 `line-height` 值，以确保所有组件的一致性。默认情况下，`background-color` 在 `body` 选择器上设置为 `#fff` 或白色。

## 标题和段落

Bootstrap 4 中标题和段落的样式没有发生重大变化。所有标题元素都被重置，移除了 `top-margin`。标题的 `margin-bottom` 值为 `0.5rem`，而段落的 `margin-bottom` 值为 `1rem`。

## 列表

列表组件有三种变体：`<ul>`、`<ol>` 和 `<dl>`。每种列表类型都移除了 `top-margin`，并具有 `1rem` 的 `bottom-margin`。如果您在列表内部嵌套列表，则完全没有 `bottom-margin`。
