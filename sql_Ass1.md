
```sql
--  ASSIGNMENT 1-------------------------------------------

-- 1
SELECT distinct p.PARTY_ID, pe.FIRST_NAME, pe.LAST_NAME, email_cm .INFO_STRING AS email, tn.CONTACT_NUMBER, p.CREATED_DATE
FROM Party AS p
JOIN Person AS pe ON p.PARTY_ID = pe.PARTY_ID
JOIN Party_Role AS pr ON pr.PARTY_ID = p.PARTY_ID AND pr.ROLE_TYPE_ID = 'CUSTOMER'
LEFT JOIN Party_Contact_Mech as e_pcm ON e_pcm.PARTY_ID = p.PARTY_ID
LEFT JOIN Contact_Mech as email_cm ON email_cm.CONTACT_MECH_ID = e_pcm.CONTACT_MECH_ID AND email_cm.CONTACT_MECH_TYPE_ID = 'EMAIL_ADDRESS'
LEFT JOIN Party_Contact_Mech as pcm_ph ON pcm_ph.PARTY_ID = p.PARTY_ID
LEFT JOIN Contact_Mech as ph_cm ON ph_cm.CONTACT_MECH_ID = pcm_ph.CONTACT_MECH_ID AND ph_cm.CONTACT_MECH_TYPE_ID = 'TELECOM_NUMBER'
LEFT JOIN Telecom_Number as tn ON tn.CONTACT_MECH_ID = ph_cm.CONTACT_MECH_ID
WHERE p.CREATED_DATE >= '2023-06-01 00:00:00' AND p.CREATED_DATE <  '2023-07-01 00:00:00';

-- 2
select pro.PRODUCT_ID, pro.PRODUCT_TYPE_ID, pro.INTERNAL_NAME
from product AS pro
join Product_Type As pt on pt.PRODUCT_TYPE_ID=pro.PRODUCT_TYPE_ID 
where  pt.IS_PHYSICAL='Y';

-- 3
select distinct GOOD_IDENTIFICATION_TYPE_ID  from good_identification gi;

select pro.PRODUCT_ID, pro.PRODUCT_TYPE_ID, pro.INTERNAL_NAME, gi.ID_VALUE as NETSUITE_ID
from product AS pro
left join GOOD_IDENTIFICATION As gi on gi.PRODUCT_ID = pro.PRODUCT_ID and gi.GOOD_IDENTIFICATION_TYPE_ID='NETSUITE_PRODUCT_ID'
where (gi.ID_VALUE IS NULL);

-- 4
select pro.PRODUCT_ID , shop.SHOPIFY_PRODUCT_ID, pro.PRODUCT_ID as HOTWAX_ID, gi.GOOD_IDENTIFICATION_TYPE_ID as ERP_ID
from product AS pro
JOIN GOOD_IDENTIFICATION As gi on gi.PRODUCT_ID = pro.PRODUCT_ID
JOIN shopify_shop_product AS shop ON pro.PRODUCT_ID = shop.PRODUCT_ID
where gi.GOOD_IDENTIFICATION_TYPE_ID="ERP_ID";

-- 5
select p.product_id,  p.PRODUCT_TYPE_ID, oh.PRODUCT_STORE_ID, sum(oi.QUANTITY) AS TOTAL_QUANTITY, p.INTERNAL_NAME, f.FACILITY_ID, oh.EXTERNAL_ID, f.FACILITY_TYPE_ID, 
ohis.ORDER_HISTORY_ID, oh.ORDER_ID, oi.ORDER_ITEM_SEQ_ID, oi.SHIP_GROUP_SEQ_ID
from ORDER_HEADER AS oh
join order_item AS oi ON oi.ORDER_ID=oh.ORDER_ID
join product as p on p.product_id=oi.product_id
join order_history as ohis on ohis.order_id= oh.order_id
join facility as f on f.FACILITY_ID=p.FACILITY_ID
where oh.status_id="ORDER_COMPLETED" and oh.order_date BETWEEN '2023-08-01 00:00:00' and '2023-08-31 23:59:59'
group by p.product_id,  p.PRODUCT_TYPE_ID, oh.PRODUCT_STORE_ID, p.INTERNAL_NAME, f.FACILITY_ID, oh.EXTERNAL_ID, f.FACILITY_TYPE_ID, 
ohis.ORDER_HISTORY_ID, oh.ORDER_ID, oi.ORDER_ITEM_SEQ_ID, oi.SHIP_GROUP_SEQ_ID;

-- 6
select oh.ORDER_ID, oh.EXTERNAL_ID as Shopify_Order_ID, oh.Grand_Total as TOTAL_AMOUNT, opp.payment_Method_Type_Id as PAYMENT_METHOD
from order_header as oh
join Order_Payment_Preference as opp on opp.ORDER_ID= oh.ORDER_ID
where order_Type_Id="SALES_ORDER" and oh.ORDER_DATE BETWEEN '2023-08-01 00:00:00' AND '2023-08-31 23:59:59';

-- 7
select oh.ORDER_ID, oh.status_id as ORDER_STATUS, opp.status_id as PAYMENT_STATUS, sh.status_id as SHIPMENT_STATUS
from order_header as oh
join Order_Payment_Preference as opp on opp.ORDER_ID= oh.ORDER_ID
join Order_Shipment as os on os.ORDER_ID=oh.ORDER_ID
join shipment as sh on sh.Shipment_Id = os.Shipment_Id
where opp.status_id= "PAYMENT_SETTLED" and sh.status_id NOT IN ("SHIPMENT_SHIPPED");

-- 8
select HOUR(os.STATUS_DATETIME) AS HOURS, COUNT(oh.ORDER_ID) AS TOTAL_ORDER
FROM order_header oh 
join order_status os on oh.ORDER_ID = os.ORDER_ID 
where os.STATUS_ID = "ORDER_COMPLETED"
group by HOURS
order by hours asc;

-- 9
select  count(oh.ORDER_ID) as TOTAL_ORDERS, sum(oh.GRAND_TOTAL) as TOTAL_REVENUE
from order_header oh
join order_item_ship_group oisp on oh.ORDER_ID = oisp.ORDER_ID
where oh.SALES_CHANNEL_ENUM_ID = "WEB_SALES_CHANNEL" 
and oh.STATUS_ID="ORDER_COMPLETED" 
and oh.ORDER_TYPE_ID = "SALES_ORDER" 
and YEAR(oh.ORDER_DATE) = 2023
and oisp.SHIPMENT_METHOD_TYPE_ID = "STOREPICKUP";   

-- 10
select COUNT(oh.ORDER_ID) AS TOTAL_ORDERS, os.CHANGE_REASON AS CANCELATION_REASON
from order_header oh
join order_status os ON oh.ORDER_ID = os.ORDER_ID
where oh.STATUS_ID = "ORDER_CANCELLED"
and os.STATUS_DATETIME >= DATE_FORMAT(NOW() - INTERVAL 1 MONTH, '24-%m-01')
and os.STATUS_DATETIME < DATE_FORMAT(NOW(), '24-%m-01')
group by os.CHANGE_REASON;

-- 11
SELECT p.PRODUCT_ID, (ii.quantity_On_Hand_Total - ii.available_To_Promise_Total) AS THRESHOLD
FROM product AS p
JOIN Inventory_Item AS ii ON ii.product_id = p.product_id;
```
