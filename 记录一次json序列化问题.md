- 原来的场景

  ```
  heimdall---dubbo--pandox
  ```

- 现在的场景

  ```
  rig---asgard---开放平台---pandox
  ```

  ```
  第一次尝试，报错如下：
  原因：同一个对象被两个对象依赖，转json
  \"posBean\":{\"$ref\":\"$.eContractSignParam.firstParty.legalPerson.posBean\"}
  
  {"code":500,"status":500,"message":"Cannot parse JSON Pointer: no valid URI fragment","data":{}} request:http://openapi.proxy.dasouche.com/v3 params:{"data":"{\"eContractSignParam\":{\"addLogo\":true,\"businessType\":0,\"elements\":{\"dianpu_shehuitongyidaima\":\"9133010009205585XF\",\"chexing\":\"2017款 高尔夫(进口) 2.0TSI R\",\"kehu_location\":\"nullnull\",\"dx_yanbaojine\":\"贰千伍佰\",\"dianpu_phone\":\"13205923599\",\"hetongriqi\":\"2019年08月15日\",\"saler_name\":\"黄斯忆\",\"yuedingticheriqi\":\"2019年05月22日\",\"kehu_phone\":\"18860116216\",\"dx_dingdanzongjia\":\"叁佰贰拾伍\",\"caiwujiesuanweikuan\":\"300.00\",\"kehu_idcard\":\"352227199212310515\",\"dx_caiwujiesuanweikuan\":\"叁佰\",\"neishi\":\"坞江黑\",\"dingdanzongjia\":\"325.00\",\"luochejia\":\"300.00\",\"dianpu_name\":\"杭州大搜车汽车服务有限公司\",\"dianpu_address\":\"123\",\"weikuan\":\"300.00\",\"dx_weikuan\":\"叁佰\",\"kehu_name\":\"雷陈杰\",\"dingdanhao\":\"S14339306521\",\"yanbaolicheng\":\"36月90000公里\",\"dx_luochejia\":\"叁佰\",\"yanbaoshangpin\":\"平安厂商延保发动机、变速箱、传动系统36月9万公里\",\"yanbaogongsi\":\"中国平安财产保险股份有限公司厦门分公司\",\"yanbaoqixian\":\"null-null\",\"saler_phone\":\"13205923599\",\"pinpai\":\"大众\",\"waiguan\":\"坞银\",\"yanbaoleixing\":\"发动机变速箱\",\"yanbaojine\":\"2500.00\",\"shangpaidi\":\"nullnull\"},\"firstParty\":{\"legalPerson\":{\"idCardNo\":\"330724197312070057\",\"name\":\"姚军红\",\"posBean\":{\"cancellingSign\":false,\"key\":\"{{gongzhang}}\",\"posType\":2},\"sign\":false,\"signType\":\"esign\",\"visible\":false},\"name\":\"杭州大搜车汽车服务有限公司\",\"number\":\"9133010009205585XF\",\"partyType\":\"other_company\",\"posBean\":{\"$ref\":\"$.eContractSignParam.firstParty.legalPerson.posBean\"},\"safeSign\":false,\"sendNotice\":false,\"signType\":\"esign\",\"visible\":true},\"imageAsync\":true,\"outCode\":\"S14339306521\",\"saveImage\":true,\"secondParty\":{\"mobile\":\"18860116216\",\"name\":\"雷陈杰\",\"number\":\"352227199212310515\",\"partyType\":\"user\",\"posBean\":{\"cancellingSign\":false,\"key\":\"{{kehuzhang}}\",\"posType\":2},\"safeSign\":true,\"sendNotice\":false,\"signType\":\"esign\",\"visible\":true},\"shopCode\":\"nm00000584\",\"singleImage\":true}}","appKey":"a3d244612d5c9ccd6ef3c23b64d570b3","api":"com.souche.pandorabox.service.econtract.EContractService#sendSms","timestamp":"1565859435"}
  
  
  第二次尝试：
  转json的时候消除对象的循环引用
  JSON.toJSONString(params, SerializerFeature.DisableCircularReferenceDetect)
  
  post request:{"data":"{\"eContractSignParam\":{\"addLogo\":true,\"businessType\":0,\"elements\":{\"dianpu_shehuitongyidaima\":\"9133010009205585XF\",\"chexing\":\"2017款 高尔夫(进口) 2.0TSI R\",\"kehu_location\":\"nullnull\",\"dianpu_phone\":\"13205923599\",\"hetongriqi\":\"2019年08月22日\",\"saler_name\":\"黄斯忆\",\"yuedingticheriqi\":\"2019年05月22日\",\"kehu_phone\":\"18768157702\",\"dx_dingdanzongjia\":\"叁佰贰拾伍\",\"caiwujiesuanweikuan\":\"300.00\",\"kehu_idcard\":\"949796464\",\"dx_caiwujiesuanweikuan\":\"叁佰\",\"neishi\":\"坞江黑\",\"dingdanzongjia\":\"325.00\",\"luochejia\":\"300.00\",\"dianpu_name\":\"杭州大搜车汽车服务有限公司\",\"dianpu_address\":\"123\",\"weikuan\":\"300.00\",\"dx_weikuan\":\"叁佰\",\"kehu_name\":\"无无\",\"dingdanhao\":\"S15613449023\",\"dx_luochejia\":\"叁佰\",\"saler_phone\":\"13205923599\",\"pinpai\":\"大众\",\"waiguan\":\"坞银\",\"shangpaidi\":\"nullnull\"},\"firstParty\":{\"legalPerson\":{\"idCardNo\":\"330724197312070057\",\"name\":\"姚军红\",\"posBean\":{\"cancellingSign\":false,\"key\":\"{{gongzhang}}\",\"posType\":2},\"sign\":false,\"signType\":\"esign\",\"visible\":false},\"name\":\"杭州大搜车汽车服务有限公司\",\"number\":\"9133010009205585XF\",\"partyType\":\"other_company\",\"posBean\":{\"cancellingSign\":false,\"key\":\"{{gongzhang}}\",\"posType\":2},\"safeSign\":false,\"sendNotice\":false,\"signType\":\"esign\",\"visible\":true},\"imageAsync\":true,\"outCode\":\"S15613449023\",\"saveImage\":true,\"secondParty\":{\"mobile\":\"18768157702\",\"name\":\"无无\",\"number\":\"949796464\",\"partyType\":\"user\",\"posBean\":{\"cancellingSign\":false,\"key\":\"{{kehuzhang}}\",\"posType\":2},\"safeSign\":true,\"sendNotice\":false,\"signType\":\"esign\",\"visible\":true},\"shopCode\":\"nm00000584\",\"singleImage\":true}}","appKey":"a3d244612d5c9ccd6ef3c23b64d570b3","api":"com.souche.pandorabox.service.econtract.EContractService#sendSms","timestamp":"1566445967"}
  
  但是业务方报错，原本是ok的业务，业务方因为拿到了两个对象，就会报错。
  ```
