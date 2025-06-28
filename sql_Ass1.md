
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
where  pt.IS_PHYSICAL='Y' ;

-- 3
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
select oi.PRODUCT_ID ,p.PRODUCT_TYPE_ID ,psc.PRODUCT_STORE_ID ,oi.QUANTITY,p.INTERNAL_NAME, oi.EXTERNAL_ID,oi.ORDER_ID,oi.ORDER_ITEM_SEQ_ID,
	oi.SHIP_GROUP_SEQ_ID,f.FACILITY_ID,f.FACILITY_TYPE_ID,oh.ORDER_HISTORY_ID  
from order_item as oi 
join order_status as os on os.ORDER_ID =oi.ORDER_ID  and  os.ORDER_STATUS_ID ="ORDER_COMPLETED" and  os.STATUS_DATETIME >'2023-08-01 00:00:00.000' and os.STATUS_DATETIME <'2023-09-01 00:00:00.000'
join product as p on oi.PRODUCT_ID =p.PRODUCT_ID
join product_store_catalog as psc  on psc.PROD_CATALOG_ID =oi.PROD_CATALOG_ID
join facility as f  on f.PRODUCT_STORE_ID =psc.PRODUCT_STORE_ID
join order_history as oh on oh.ORDER_ID =oi.ORDER_ID;

-- 6
select oh.ORDER_ID, oh.EXTERNAL_ID as Shopify_Order_ID, oh.Grand_Total as TOTAL_AMOUNT, opp.payment_Method_Type_Id as PAYMENT_METHOD
from order_header as oh
join Order_Payment_Preference as opp on opp.ORDER_ID= oh.ORDER_ID
where order_Type_Id="SALES_ORDER" and oh.ORDER_DATE BETWEEN '2023-08-01 00:00:00' AND '2023-08-31 23:59:59';

-- 7
select distinct oh.ORDER_ID, oh.status_id as ORDER_STATUS, opp.status_id as PAYMENT_STATUS, ss.status_id as SHIPMENT_STATUS
from order_header as oh
left join Order_Payment_Preference as opp on opp.ORDER_ID= oh.ORDER_ID
LEFT JOIN order_shipment AS os ON os.ORDER_ID = oh.ORDER_ID
LEFT JOIN shipment_status AS ss ON ss.SHIPMENT_ID = os.SHIPMENT_ID
where opp.STATUS_ID = 'PAYMENT_SETTLED' AND (ss.STATUS_ID IS NULL OR ss.STATUS_ID != 'SHIPMENT_SHIPPED');

-- 8
select HOUR(os.STATUS_DATETIME) AS HOURS, COUNT(oh.ORDER_ID) AS TOTAL_ORDER
FROM order_header oh 
join order_status os on oh.ORDER_ID = os.ORDER_ID 
where os.STATUS_ID = "ORDER_COMPLETED"
group by HOURS
order by hours asc;

-- 9
select  count(distinct oh.ORDER_ID) as TOTAL_ORDERS, sum(distinct oh.GRAND_TOTAL) as TOTAL_REVENUE
from order_header oh
join Shipment as ship on ship.PRIMARY_ORDER_ID = oh.ORDER_ID
where oh.STATUS_ID="ORDER_COMPLETED" 
and YEAR(oh.ENTRY_DATE) = YEAR(CURDATE()) - 1
and ship.SHIPMENT_METHOD_TYPE_ID = "STOREPICKUP";     

-- 10
select COUNT(oh.ORDER_ID) AS TOTAL_ORDERS, os.CHANGE_REASON AS CANCELATION_REASON
from order_header oh
join order_status os ON oh.ORDER_ID = os.ORDER_ID
where oh.STATUS_ID = "ORDER_CANCELLED"
AND MONTH(os.STATUS_DATETIME) = MONTH(CURRENT_DATE())-1
group by os.CHANGE_REASON;

-- 11
select distinct pf.PRODUCT_ID, pf.minimum_stock AS THRESHOLD
from product_facility as pf
join facility f on f.FACILITY_ID =pf.FACILITY_ID and f.FACILITY_TYPE_ID = 'CONFIGURATION';
```
