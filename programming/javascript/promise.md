



```javascript
 await fetch(base + '/api/keystone/update_user_login_info', {
     method: 'post',
     body: JSON.stringify({
         "user_name": username,
         "update_type": "last_login"
     }),
     headers: {
         'Content-Type': 'application/x-www-form-urlencoded',
         'Bihu-Token': autoTestToken
     }
 }).then(function(data){
     return data.text();
 }).then(ret => {
     var resp = JSON.parse(ret); 
     assert.ok(resp.code===200);
     assert.ok(resp.kind==='success');
     print('\t修改成功')
 });

```

