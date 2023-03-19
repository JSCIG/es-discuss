# range 提案的浮点数问题

目前 range 提案支持浮点数，我在 https://github.com/tc39/proposal-Number.range/issues/57 中简要地提到了浮点数的精度问题：

> it's rare and always have precision-related issue to iterating floats, 
> so many programming languages do not support iterating floats in the core lib

在上周的 JSCIG 会议上我们再次讨论了这个问题，Jack 认为这是一个值得进一步考虑的问题，建议我单独开个issue。

简单说，浮点数总是存在精度问题。

比如 `range(0, 10, 3)` 产生 `0, 3, 6, 9`，而 `range(0, 1, 0.3)` 产生 `0, 0.3, 0.6, 0.8999999999999999`。这里存在几个问题：

第一，无论采用累加法还是乘法，都可能产生精度丢失。

第二，由于精度丢失，导致最后一个值可能正好略小于 end 而被纳入结果。比如 `range(0, 0.9, 0.3)` 得到4个数。

第三，精度问题是偶发的，也就是说，很容易产生开发者觉得正确但当参数稍作变化就踩坑的情形。比如`range(100, x, 0.3)`当x是小于166的整数时都没有问题，直到166出现问题。

注意，我认为并不能简单地将浮点数精度问题视作开发者教育问题，应该反过来考虑到底在什么情况下用户要使用浮点数，如果没有合理的浮点数的range的用例，也就意味着使用浮点数在实践中几乎总是产生非预期的效果。最终MDN等文档不得不教育开发者避免在range中使用浮点数。另一方面，目前的工具链（如TS或ESLint）并没有足够能力在开发时提示开发者不当引入了浮点数，也就是说在实际工程中我们无法提供保护。与其如此，还不如一开始就不要支持浮点数。

另一方面，那么我们是否有足够solid的浮点数的range用例呢？以我个人经验看，似乎是很可疑的。

比如在`range(0, 0.9, 0.3)`的例子中，我估计大部分开发者会预期得到`0, 0.3, 0.6`的结果，也就是说，开发者实际想要的并不是浮点数，而是定点数或者decimal。在没有decimal之前，此类用例最合理的方式是写成 `range(0, 9, 3).map(x => x / 10)`。

确实可能也有一些用例要使用浮点数且不在意精度，比如绘制图表时，我们在x轴的某个区间取样若干个点，计算这若干个点对应的y值并绘制。但在这样的用例中，更直接的方式是指定区间的取样数量而不是指定step。所以更符合需求的是类似于 [numpy 提供的 linspace 函数](https://numpy.org/doc/stable/reference/generated/numpy.linspace.html)的API。我们当然也可以通过 `range(0, sampleCount).map(i => start + (end - start) * i / sampleCount)` 来达到目标，而此方式中`range`也无需支持浮点数。

最后，如果我们不支持浮点数，也可以避免目前草案中针对infinity之类情况做特殊处理，也避免产生safeint范围之外的奇异行为（迭代产生重复的数字）。

以上。
