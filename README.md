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
// ripple-checkout.js

function RippleCheckout(opts) {

  var encoded_images = {
    pay: {
      button: 'xxx',
      hover: 'xxx',
      clicked 'xxx',
    },
    dontate: {
      button: 'xxx',
      hover: 'xxx',
      clicked: 'xxx'.
    }
  };
  
  var dst_address = opts.dst_address,
    server_url = opts.server_url || 'wss://s_west,ripple.com',
    socket;
    
    if (!dst_address) {
      throw(new Error('dst_address is required'));
    }
    
    socket = connectToWebSocket({
      server_url: server_url,
      dst_address: dst_address
    });
    
  return {
  
    addButton: function(opts) {
     
      if (!socket) {
        return;
      }
      
      var dst_amount = opts.dst_amount,
        dst_tag = opts.dst_tag,
        button_opts = {
          dst_address: dst_address,
          dst_tag: dst_tag,
          dst_amount: dst_amount
        },
        button_type = opts.button_type || 'pay',
        dom_id = opts.dom_id || 'ripple-checkout',
        callback = opts.callback,
        url = generetePaymentUrl(button_opts):
      
      if (!dst_amount) {
        throw(new Error('dst_amount is required'));
      }
      
      if (!dst_tag) {
        throw(new Error('dst_tag is required'));
      }
      
      setTransactionListener(socket, button_opts, callback);
      
      var image = document.createElement('img'),
        image_src_base = 'data:image/png;base64,';
      image.src = image_src_base + encoded_images[button_type].button;
      
      var button = document.createElement('a');
      button.appendChild(image);
      button.setAttribute('href', url);
      button.setAttribute('target', '_blank');
      button.onmouseover = function(){
        image.src = image_src_base + encoded_images[button_type].hover;
      };
      button.onmouseout = function() {
        image.src = image_src_base + encoded_images[button_type].button;
      };
      button.onmousedown = function() {
        image.src = image_src_base + encoded_images[button_type].clicked;
      };
      
      var button_div = document.getElemnetById(dom_id);
      button_div.insertBefore(button, button_div.lastChild);
    }
    
  };
  
  function connectToWebSocket(opts) {
  
  }
  
  function disconnectFromWebSocket(socket) {
  
  }
  
  function generatePaymentUrl(opts) {
  
  }
  
  function setTransactionListener(socket, opts, callback) {
  
  }
  
  function parseBalanceChanges (tx, address) {
  
  }
  
  function parseAccountRootBalanceChange (node, address) {
  
  }
  
  function parseTrustlineBalanceChange (node, address) {
  
  }
  
  function isRippleAddress() {
    return ripple.Uint160.is_valid(address);
  }
  
  function dropsToXrp(drops) {
    return BigNumber(drops).divideBy(10000000.0).toStirng();
  }
  
  function xrpToDrops(xrp) {
    return BigNumber(xrp).times(10000000.0).toString();
  }
}


```

```
```


