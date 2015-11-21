get domain id
=============
```shell
curl -X POST https://dnsapi.cn/Domain.Info -d 'login_token=11409,9447ebc4f003f108813bc3224f7ff56d&format=json&domain=cclouis.com'
```

```json
{
  "status": {
    "code": "1",
    "message": "Action completed successful",
    "created_at": "2015-11-20 23:11:12"
  },
  "domain": {
    "id": "17642360",
    "name": "cclouis.com",
    "punycode": "cclouis.com",
    "grade": "DP_Free",
    "grade_title": "新免费套餐",
    "status": "enable",
    "ext_status": "",
    "records": "15",
    "group_id": "1",
    "is_mark": "no",
    "remark": false,
    "is_vip": "no",
    "searchengine_push": "yes",
    "user_id": "935423",
    "created_on": "2014-08-04 21:06:29",
    "updated_on": "2014-12-17 14:21:06",
    "ttl": "600",
    "cname_speedup": "disable",
    "owner": "dl_snaker@hotmail.com"
  }
}
```


get domain info
===============
```shell
curl -X POST https://dnsapi.cn/Record.List -d "login_token=11409,9447ebc4f003f108813bc3224f7ff56d&format=json&domain_id=17642360&sub_domain=pi"
```

```json
{
  "status": {
    "code": "1",
    "message": "Action completed successful",
    "created_at": "2015-11-20 23:18:33"
  },
  "domain": {
    "id": 17642360,
    "name": "cclouis.com",
    "punycode": "cclouis.com",
    "grade": "DP_Free",
    "owner": "dl_snaker@hotmail.com"
  },
  "info": {
    "sub_domains": "15",
    "record_total": "1"
  },
  "records": [
    {
      "id": "124541291",
      "name": "pi",
      "line": "默认",
      "type": "A",
      "ttl": "600",
      "value": "114.245.255.188",
      "weight": null,
      "mx": "0",
      "enabled": "1",
      "status": "enabled",
      "monitor_status": "",
      "remark": "",
      "updated_on": "2015-10-25 01:12:20",
      "use_aqb": "no"
    }
  ]
}
```

update record
=============

```
curl -X POST https://dnsapi.cn/Record.Remove -d 'login_token=11409,9447ebc4f003f108813bc3224f7ff56d&format=json&domain_id=17642360&record_id=128868915'

{"status":{"code":"1","message":"Action completed successful","created_at":"2015-11-20 23:41:16"}}


curl -X POST https://dnsapi.cn/Record.Create -d 'login_token=11409,9447ebc4f003f108813bc3224f7ff56d&format=json&domain_id=17642360&sub_domain=pi&record_type=A&value=8.8.4.4&record_line=默认'

{"status":{"code":"1","message":"Action completed successful","created_at":"2015-11-20 23:41:16"},"record":{"id":"128869459","name":"pi","status":"enabled","weight":null}}
```