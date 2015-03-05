生产环境:
IP: 124.202.147.6
tomcat: /usr/local/apache-tomcat...
nginx: /usr/local/nginx
jdk: /usr/local/jdk..
mysql: /etc/my.conf 

静态资源: /home/data/fengke
日志: /home/log4j/..
war包: /home/war/fengke.war

布署命令: 
开始:fengke start
停止:fengke stop
重启:fengke restart
布署:fengke deploy(会拷贝 /home/war/fengke.war 到 /usr/local/apache-tomcat/webapps/ROOT.war 并重启tomcat)


订单信息：
订单编号：order.sn
产品名称：orderItem.fullName
订单来源：order.category
订单金额：order.amount
会员名：order.member.username
方式：
支付方式：order.paymentMethodName
订单状态：order.orderStatus
配送状态：order.shippingStatus
创建日期：order.createDate
商品数量：order.quantity（无法获得）
是否促销：order.promotion
促销折扣：order.promotionDiscount
调整金额：order.offsetAmount
运费：order.freight
手机号码：order.selectedPhone
入网城市：order.networkCity
号码类型：neusoftPhoneNumber.type
号码预存话费金额：neusoftPhoneNumber.beforehandFee
开户预存金额：neusoftPhoneNumber.prettyNumFeature.prettyNumLevel.openFee
合约类型：order.phonePackage.contractType
月承诺最低消费：neusoftPhoneNumber.prettyNumFeature.prettyNumLevel.minFee
要求在网时长：neusoftPhoneNumber.prettyNumFeature.prettyNumLevel.onlineTime
机主姓名：order.phoneName
身份证号：order.idCard
身份证地址：order.idCardAddress
运单号：shipping.trackingNo
收货人：order.consignee
收货地址：order.address
收货人联系电话：order.phone
备注：order.memo


订单编号	产品名称	订单来源	订单金额	会员名	方式	支付方式	
订单状态	配送状态	创建日期	商品数量	是否促销	促销折扣	调整金额	
运费	手机号码	入网城市	号码类型	号码预存话费金额	开户预存金额	
合约类型	月承诺最低消费	要求在网时长	机主姓名	身份证号	身份证地址	运单号	收货人	收货人地址	收货人联系电话	备注

//查询order订单表中已有数据
select
 t.sn as 订单编号,
 t.selected_phone 手机号码,
 decode(t.category,0,'风客旅行',1,'微信商城',2,'app') 订单来源,
 t.amount_paid 订单金额 ,
 m.username as 会员名,
 decode(p.method,0,'在线支付',1,'线下支付',2,'预存款支付') 方式,
 p.payment_method as 支付方式,
 decode(t.order_status,0,'未确认',1,'已确认',2,'已完成',3,'已取消') as 订单状态,
 decode(t.shipping_status,0,'未发货',1,'部分发货',2,'已发货',3,'部分退货',4,'已退货') as 配送状态,
 t.create_date as 创建日期, 
 t.promotion as 是否促销,
 t.promotion_discount 促销折扣,
 t.offset_amount 调整金额 ,
 t.freight 运费,
 t.network_city 入网城市 ,
 t.phone_name 机主姓名,
 t.id_card 身份证号,
 t.id_card_address 身份证地址,
 t.consignee 收货人,
 t.address 收货地址,
 t.phone 收货人联系电话,
 t.memo 备注
  from 
  xx_order t,
  xx_member m,
  xx_payment p 
   where
    t.member=m.id and p.orders=t.id and t.payment_status=2 and p.status=1 and
    to_char(t.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');
 
--产品名称
//查询订单子项以及商品的名称   
select o.sn 订单编号,oi.product 商品ID, oi.full_name 商品全称 from xx_order o left join xx_order_item oi  on o.id = oi.orders left join xx_payment p on o.id=p.orders 
  where p.status=1 and o.category=0 and to_char(o.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');
 
//如果商品为秒关爱套包，查询秒关爱套包商品所关联的商品  
select pp.product 套包商品ID, pp.products 套包关联商品ID,p.full_name  from xx_package_product pp left join xx_product p  on pp.products= p.id where pp.product in(select oi.product 商品ID
  from xx_order o left join xx_order_item oi  on o.id = oi.orders left join xx_payment p on o.id=p.orders left join xx_product pd on oi.product=pd.id   
  where p.status=1 and o.category=0 and pd.combination_type=0 and  
    to_char(p.payment_date,'yyyy-mm-dd')<=to_char(sysdate-1,'yyyy-mm-dd')) ;

--查询秒关爱订单关联的手机号id
select t.sn 订单编号, xon.orders 订单ID,xon.neusoft_phone_numbers 手机号ID from  xx_order_neusoftphonenumber xon  left join xx_order t on t.id = xon.orders ;
select npn.id,npn.service_id 手机号,npn.id_card 身份证号,npn.name 用户姓名,npn.id_address 身份证地址  from neusoft_phone_number npn ;



//运单号查询
select t.sn 订单号,t.selected_phone 手机号码,s.tracking_no 运单号 
from xx_shipping s,xx_order t 
where s.orders=t.id and t.payment_status=2 and t.shipping_status=2 
//合约类型
select t.sn 订单号,decode(pp.contract_type,0,'12个月',1,'24个月') 合约类型 
from  xx_phone_package pp,xx_order t,xx_payment pm 
where pp.id=t.phone_package 
and pm.orders=t.id 
and pm.status=1 and t.phone_package is not null 
and to_char(t.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');
//有关选号（除妙关爱订单）的订单查询 根据订单表中手机号对应
select
 n.service_id 手机号,
 decode(n.type,0,'中国移动',1,'中国联通',2,'中国电信') 号码类型 ,
 n.beforehand_fee 号码预存话费金额,
 pnl.open_fee 开户预存金额,
 pnl.min_fee 月承诺最低消费,
 decode(pnl.online_time,0,'不限',1,'12个月',2,'24个月') 要求在网时长
 from 
  neusoft_phone_number n,
  xx_pretty_num_feature pnf,
  xx_pretty_num_level pnl
   where
    n.pretty_num_feature=pnf.id and
    pnf.pretty_num_level=pnl.id and
   n.service_id in(select t.selected_phone from xx_order t ,xx_payment p
                where p.orders=t.id and p.status=1 and to_char(t.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd') 
                and t.selected_phone is not null
                );
//妙关爱订单号码信息查询
select
 n.type 号码类型 ,
 n.beforehand_fee 号码预存话费金额,
 pnl.open_fee 开户预存金额,
 pnl.min_fee 月承诺最低消费,
 pnl.online_time 要求在网时长,
 n.id_card 身份证号码,
 n.name 机主姓名,
 n.id_address 身份证地址 
 from 
  neusoft_phone_number n,
  xx_pretty_num_feature pnf,
  xx_pretty_num_level pnl
   where
    n.pretty_num_feature=pnf.id and
    pnf.pretty_num_level=pnl.id and
    n.id in(select onpn.neusoft_phone_numbers from xx_order_neusoftphonenumber onpn where onpn.orders in(select t.id from xx_order t 
                where  to_char(t.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd')));



退款管理（完成）
支付方式	退款金额	收款人	创建日期	退款银行	备注

select r.payment_method 支付方式, r.amount 退款金额 ,r.payee 收款人,r.create_date 创建日期,r.bank 退款银行,r.memo 备注 from xx_refunds r where to_char(r.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');

退货管理（完成）
订单编号	发货人	创建日期	退货产品	退货金额（表中无退货金额）	退货状态（没有对应状态数据）	备注
select r.sn 订单编号,r.shipper 发货人,r.create_date 创建日期,ri.name 退货产品,r.memo 备注 from xx_returns r left join xx_returns_item ri on ri.returns=r.id where to_char(r.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');

充值管理（完成）
订单编号	充值结果	支付金额	手机号码	充值卡号	订单状态	备注(该表没备注项)

select xq.sn 订单编号 ,
decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 充值结果,
 xq.payment_amount 支付金额,
 decode(xq.type,0,'充值卡',1,'网银') 充值类型,
 xq.phone 手机号码,
 xq.card_no 充值卡号,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 订单状态 
from xx_charge_log xq left join xx_payment p on xq.id=p.charge_log  
where xq.status=1 and p.status=1 and to_char(xq.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');
//充值卡充值
select xq.sn 订单编号 ,
decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 充值结果,
 xq.payment_amount 支付金额,
 decode(xq.type,0,'充值卡',1,'网银') 充值类型,
 xq.phone 手机号码,
 xq.card_no 充值卡号,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 订单状态 
from xx_charge_log xq  
where xq.status=1 and xq.card_no is not null and to_char(xq.create_date,'yyyy-mm-dd')=to_char(sysdate-1,'yyyy-mm-dd');



    
 -----财务报表
--订单主表

select
 p.payment_date 交易时间,
 t.sn as 风客旅行订单号,
 p.sn 后台收款单号,
 p.amount 交易金额 ,
 t.freight 运费,
 p.amount-t.freight 商品金额,
 decode(t.shipping_status,0,'未发货',1,'部分发货',2,'已发货',3,'部分退货',4,'已退货') as 配送状态, 
 p.payment_method as 支付方式,
 t.network_city 地区 
  from 
  xx_order t,
  xx_payment p
   where
    t.id=p.orders and p.status=1 and t.category=0 and 
    p.payment_date>to_date('2014-10-20 23:59:59','yyyy-mm-dd hh24:mi:ss')and 
    to_char(p.payment_date,'yyyy-mm-dd')<=to_char(sysdate,'yyyy-mm-dd')

 --订单包含商品子表  
    select oi.product 商品ID, oi.full_name 商品全称,oi.price 商品价格,o.sn 风客旅行订单编号 
  from xx_order o left join xx_order_item oi  on o.id = oi.orders left join xx_payment p on o.id=p.orders 
  where p.status=1 and o.category=0 and 
    p.payment_date>to_date('2014-10-20 23:59:59','yyyy-mm-dd hh24:mi:ss')and 
    to_char(p.payment_date,'yyyy-mm-dd')<=to_char(sysdate,'yyyy-mm-dd')
	--订单中妙关爱套包子商品
  select pp.product 套包商品ID, pp.products 套包关联商品ID,p.full_name 商品全称 ,pp.preferential_price 商品价格
  from xx_package_product pp left join xx_product p  on pp.products= p.id where pp.product in(select oi.product 商品ID
  from xx_order o left join xx_order_item oi  on o.id = oi.orders left join xx_payment p on o.id=p.orders left join xx_product pd on oi.product=pd.id   
  where p.status=1 and o.category=0 and pd.combination_type=0 and 
    p.payment_date>to_date('2014-10-20 23:59:59','yyyy-mm-dd hh24:mi:ss')and 
    to_char(p.payment_date,'yyyy-mm-dd')<=to_char(sysdate,'yyyy-mm-dd'))

---充值订单
//网上充值
select p.payment_date 交易时间, xq.sn 风客旅行订单编号 ,p.sn 后台收款单号,xq.payment_amount 支付金额,
xq.card_no 充值卡号,xq.phone 手机号码,
(select distinct decode(ba.type, 0, '中国移动', 1,'中国联通',2,'中国电信')
          from boss_area ba
         where ba.no_prefix is not null
           and xq.phone like ba.no_prefix || '%' ) 运营商,
 (select ba.name
          from boss_area ba
         where ba.no_prefix is not null
           and xq.phone like ba.no_prefix || '%' ) 号码归属地,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 类型,
 p.payment_method as 支付方式 
from xx_charge_log xq left join xx_payment p on xq.id=p.charge_log where xq.status=1 and p.status=1
and p.payment_date>to_date('2014-10-20 23:59:59','yyyy-mm-dd hh24:mi:ss') and 
 to_char(p.payment_date,'yyyy-mm-dd')<=to_char(sysdate,'yyyy-mm-dd');
 //充值卡充值
 select xq.create_date 交易时间, xq.sn 风客旅行订单编号 ,xq.payment_amount 支付金额,
xq.card_no 充值卡号,xq.phone 手机号码,
(select distinct decode(ba.type, 0, '中国移动', 1,'中国联通',2,'中国电信')
          from boss_area ba
         where ba.no_prefix is not null
           and xq.phone like ba.no_prefix || '%' ) 运营商,
 (select ba.name
          from boss_area ba
         where ba.no_prefix is not null
           and xq.phone like ba.no_prefix || '%' ) 号码归属地,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 类型 ,
 decode(xq.type,0,'高阳捷讯',1,'网银') 支付方式
from xx_charge_log xq  where xq.status=1 and xq.card_no is not null
and xq.create_date>to_date('2014-10-20 23:59:59','yyyy-mm-dd hh24:mi:ss') and 
 to_char(xq.create_date,'yyyy-mm-dd')<=to_char(sysdate,'yyyy-mm-dd');
--end---  
 select xq.sn 订单编号 , xq.payment_amount 支付金额,xq.status 订单支付状态,xq.create_date 创建日期  from xx_charge_log xq where xq.status=1;
select
 t.sn as 订单编号,
 t.amount_paid 订单金额 ,
  t.freight 运费,
 t.payment_status 订单支付状态,
 t.create_date as 创建日期, 
 t.area_name 归属地
  from 
  xx_order t
   where
    t.payment_status=2;   
   
 --运营按日期导出充值数据
 select xq.sn 订单编号 ,
decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 充值结果,
 xq.payment_amount 支付金额,
 decode(xq.type,0,'充值卡',1,'网银') 充值类型,
 xq.phone 手机号码,
 xq.card_no 充值卡号,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 订单状态 
from xx_charge_log xq left join xx_payment p on xq.id=p.charge_log  
where xq.status=1 and p.status=1 and 
p.payment_date>to_date('2014-11-03 23:59:59','yyyy-mm-dd hh24:mi:ss') and
 p.payment_date<to_date('2014-11-05 23:59:59','yyyy-mm-dd hh24:mi:ss') ;  


select xq.sn 订单编号 ,
decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 充值结果,
 xq.payment_amount 支付金额,
 decode(xq.type,0,'充值卡',1,'网银') 充值类型,
 xq.phone 手机号码,
 
 xq.card_no 充值卡号,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 订单状态 
from xx_charge_log xq  
where xq.status=1 and xq.card_no is not null and xq.create_date>to_date('2014-11-03 23:59:59','yyyy-mm-dd hh24:mi:ss') and
 xq.create_date<to_date('2014-11-05 23:59:59','yyyy-mm-dd hh24:mi:ss');
   
    
    
//财务数据导出一个月的  
    select p.payment_date 交易时间, xq.sn 风客旅行订单编号 ,p.sn 后台收款单号,xq.payment_amount 支付金额,
xq.card_no 充值卡号,xq.phone 手机号码,
(select distinct decode(ba.type, 0, '中国移动', 1,'中国联通',2,'中国电信')
          from boss_area ba
         where ba.no_prefix is not null
           and xq.phone like ba.no_prefix || '%' ) 运营商,
 (select ba.name
          from boss_area ba
         where ba.no_prefix is not null
           and xq.phone like ba.no_prefix || '%' ) 号码归属地,
 decode(xq.status,0,'未支付',1,'充值成功',2,'充值失败') 类型,
 p.payment_method as 支付方式 
from xx_charge_log xq left join xx_payment p on xq.id=p.charge_log where xq.status=1 and p.status=1
and p.payment_date>to_date('2014-10-20 23:59:59','yyyy-mm-dd hh24:mi:ss') and 
 p.payment_date<=to_date('2014-11-01 00:00:00','yyyy-mm-dd hh24:mi:ss');
    
    
2014-11-18 添加管理员权限    已执行

insert into xx_role_authority (role,authorities) values(1,'admin:chargelog');
insert into xx_role_authority (role,authorities) values(1,'admin:app');
insert into xx_role_authority (role,authorities) values(1,'admin:buchong');  
insert into xx_role_authority (role,authorities) values(1,'admin:singleLogin');
commit;
    
    
