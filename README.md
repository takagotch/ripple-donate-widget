### ripple-donate-widget
---
https://github.com/emschwartz/ripple-donate-widget

```js
var checkout = RippleCheckout({
  dst_address: 'xxxx',
  server_url: 'wss://s_west.ripple.com'
}):

checkout.addButton({
  dst_amount: {
    value: '10',
    currency: 'XRP',
    issuer: ''
  },
  dst_tag: Math.floor(Math.random() * 00000),
  callback: function(err, confirmed_transaction) {
    if (err) {
      alert('Error: \n\n' + err);
    } else {
      alert('Transaction Confirmed: \n\n' + JSON.stringify(confirmed_transaction));
    }
  }
});
```

```js
//
```

```
```


