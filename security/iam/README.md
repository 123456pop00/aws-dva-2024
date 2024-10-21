# IAM

{% hint style="info" %}
**Good to know:** you can embed a Storybook canvas by simple pasting the canvas link and hitting enter.
{% endhint %}

An Icon is a piece of visual element, but we must ensure its accessibility while using it. It can have **2 purposes**:

* _**decorative only**_: for example, it illustrates a label next to it. We must ensure that it is ignored by screen readers, by setting aria-hidden attribute (ex: `<Icon icon="check" aria-hidden />`)
* _**non-decorative**_: it means that it delivers information. For example, an icon as only child in a button. The meaning can be obvious visually, but it must have a proper text alternative via `aria-label` for screen readers. (ex:`<Icon icon="print" aria-label="Print this document" />`)
