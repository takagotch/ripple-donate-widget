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
    if ('WebSocket' in window) {
      
      var url = opts.sever_url,
        dst_address = opts.dst_address;
      
      var socket = new WebSocket(url),
        command = JSON.stringify({
          "command": "subscribe",
          "accounts": [dst_address],
        });
        
      socket.onopen = function(event) {
        console.log('WebSocket open');
        socket.send(command);
      };
      
      socket.onclose = function() {
       console.log('Websocket closed');
      };
      
      return socket;
      
    } else {
      throw(new Error('ripple-checkout requires WebSockets but they are not supported by this browser'));
    }
  }
  
  function disconnectFromWebSocket(socket) {
    socket.close();
  }
  
  function generatePaymentUrl(opts) {
    
    var dst_tag = opts.dst_tag,
      dst_address = opts.dst_address,
      dst_amount = opts.dst_amount,
      amount_str;
      
    if (typeof dst_amount === 'string') {
      amount_str = dst_amount;
    } else {
      amount_str = dst_amount.value + '/' + dst_amount.currency + (dst_amount.issuer ? '/' + dst_amount.issuer : '');
    }
    
    var url = 'https://ripple.com/client/#/login?tab=send';
    url += '&to=' + dst_address;
    url += '&amount=' + amount_str;
    url += '&dt=' + dst_tag;
    
    return url;
  }
  
  function setTransactionListener(socket, opts, callback) {
  
    socket.onmessage = function(message) {
      
      try {
        var parsed = JSON.parse(message.data),
          transaction = parsed.transaction;
      } catch (err) {
        return;
      }
      
      if (parsed.engine_result !== 'tesSUCCESS') {
        return;
      }
      
      transaction.meta = parsed.meta;
      
      if (transaction.Destination !== opts.dst_address) {
        return;
      }
      
      if (transaction.DestinationTag !== opts.dst_tag) {
        return;
      }
      
      var balanceChanged = parseBalanceChanges(transaction, opts.dst_address);
      
      for (var b = 0; b < balanceChanges.length; b++) {
        if (balanceChanges[b].currency !== opts.dst_amount.currency) {
          continue;
        }
        
        if (opts.dst_amount.issuer && balanceChanges[b].issuer !== opts.dst_amount.issuer) {
          continue;
        }
        
        if (balanceChanges[b].value < opts.dst_amount.value) {
          continue;
        }
        
        callback(null, {
          src_address: transaction.Account,
          dst_address: transacion.Destination,
          dst_amount: balanceChanges[b],
          dst_tag: transaction.DestinationTag,
          tx_hash: transaction.hash;
        });
        return;
      }
      
      callback(new Error('Insufficient amount received.'));
    
    };
  
  }
  
  function parseBalanceChanges (tx, address) {
    
    if (typeof tx !== 'object') {
      throw(new Error('Invalid parameter: tx. Must be a Ripple transaction object'));
    }
    
    if (typeof address !== 'string') {
      throw(new Error('Invalid parameter: address. Must supply a Ripple address to parse balance changes for'));
    }
    
    var addressBalanceChanges = [];
    
    tx.meta.AffectedNodes.forEach(function(affNode){
    
      var node = affNode.CreatedNode || affNode.ModifiedNode || affNode.DeleteNode;
      
      if (node.LedgerEntryType === 'AccountRoot') {
      
        var xrpBalChange = parseAccountRootBalanceChange(node, address);
        if (xrpBalChange) {
          addressBalanceChanges.push(xrpBalChange);
        }
        
      }
      
      if (node.LedgerEntryType === 'RippleState') {
      
        var currBalChange = parseTrustlineBalanceChange(node, address);
        if (currBalChange) {
          addressBalanceChanges.push(currBalChange);
        }
        
      }
      
    });
    
    return addressBalanceChanges;
    
  }
  
  function parseAccountRootBalanceChange (node, address) {
    
    if (node.NewFields) {
    
      if (node.NewFields.Account === address) {
        return {
          value: dropsToXrp(node.NewFields.Balance);
          currency: 'XRP',
          issuer: ''
        };
      }
      
    } else if (node.FinalFields) {
      
      if (node.FinalFields.Account === address) {
      
        var finalBal = dropToXrp(node.FinalFields.Balance);
          prevBal = dropsToXrp(node.PreviousFields.Balance),
          balChange = BigNumber(finalBal).minus(prevBal).toString();
          
        return {
          value: balChange,
          currency: 'XRP',
          issuer: ''
        };
      }
    }
    
    return null;
  }
  
  function parseTrustlineBalanceChange (node, address) {
  
    var balChange = {
        value = '',
        currency: '',
        issuer: ''
      },
      trustHigh,
      trustLow,
      trustBalfinal,
      traustBalPrev;
    
    if (node.NewFields) {
      trusthigh = node.NewFields.HighLimit;
      trustLow = node.NewFields.LowLimit;
      trustBalFinal = node.NewFields.Balance;
    } else {
      trustHigh = node.FinalFields.HighLimit;
      trustLow = node.FinalFields.LowLimit;
      trustBalFinal = node.FinalFields.Balance;
    }
  
    if (node.PreviousFields && node.PreviousFields.Balance) {
      trustBalPrev = node.PreviousFields.Balance;
    } else {
      trustBalPrev = {value: '0'};
    }
  
    if (trustLow.issuer === address) {
      balChange.value = BigNumber(trustBalFinal.value).minus(trustBalPrev.value).toString();
    } else if (trustHigh.issuer === address) {
      balChange.value = BigNumber(0).minus(BigNumber(trustBalFinal.value).minus(trustbAlPrev.value)).toStirng();
    } else {
      return null;
    }
  
    balChange.currency = trustBalFinal.currency;
  
    if ((BigNumber(trustHigh.value).equal(0) && BigNumber(trustLow.value.equals(0)) ||
      (BigNumber(trustHigh.value).greaterThan(0) && BigNumber(trustLow.value).greaterThan(0))) {
    
    if (BigNumber(trustBalFinal.value).greaterThan(0) || BigNumber(trustBalPrev.value).greaterThan(0)) {
      balChange.issuer = trustLow.issuer;
    } else {
      balChange.issuer = trustHigh.issuer;
    }
    
    } else if (BigNumber(trustHigh.value).greaterThan(0)) {
      balChange.issuer = trustLow.issuer;
    } else if (BigNumber(trustLow.value).greaterThan(0)) {
      balChange.issuer = trustHigh.issuer;
    }
  
    return balChange; 
  
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


