-- ASSIGNMENT 2-------------------------------------------

-- 1
select DISTINCT oh.ORDER_ID, orole.PARTY_ID as Customer_ID, per.first_name as CUSTOMER_NAME , pa.Address1 as STREET_ADDRESS, pa.CITY, pa.State_Province_Geo_Id as STATE_PROVINCE,
pa.Postal_Code , tn.COUNTRY_CODE , oh.status_id as ORDER_STATUS, oh.ORDER_DATE
from order_header as oh 
join order_Role as orole on oh.ORDER_ID = orole.ORDER_ID
join person as per on per.party_id = orole.party_id
join order_contact_mech as ocm on ocm.ORDER_ID= oh.ORDER_ID and ocm.CONTACT_MECH_PURPOSE_TYPE_ID='SHIPPING_LOCATION'
join order_contact_mech as oc on oh.ORDER_ID = oc.ORDER_ID AND oc.CONTACT_MECH_PURPOSE_TYPE_ID = "PHONE_SHIPPING"
join telecom_number tn on oc.CONTACT_MECH_ID = tn.CONTACT_MECH_ID 
join postal_address as pa on ocm.CONTACT_MECH_ID = pa.CONTACT_MECH_ID
join order_status  os on oh.ORDER_ID = os.ORDER_ID
where oh.order_type_id = 'SALES_ORDER' and os.STATUS_DATETIME BETWEEN '2023-10-01 00:00:00' AND '2023-10-31 23:59:59' 
and oh.status_id  IN ('ORDER_COMPLETED','ORDER_CREATED');

-- 2
select DISTINCT oh.ORDER_ID, per.first_name,per.last_name, pa.Address1 as STREET_ADDRESS, pa.CITY, pa.State_Province_Geo_Id as STATE_PROVINCE,
pa.Postal_Code , oh.status_id as ORDER_STATUS, oh.ORDER_DATE, oh.grand_Total as Total_Amount
from order_header as oh
join order_Role as orole on oh.ORDER_ID = orole.ORDER_ID
join person as per on per.party_id = orole.party_id
join order_contact_mech as ocm on ocm.ORDER_ID= oh.ORDER_ID 
join contact_mech cm on ocm.CONTACT_MECH_ID = cm.CONTACT_MECH_ID
join postal_address as pa on cm.CONTACT_MECH_ID = pa.CONTACT_MECH_ID
where pa.STATE_PROVINCE_GEO_ID = "NY";

-- 3
select p.PRODUCT_ID, p.INTERNAL_NAME, sum(oi.quantity)as TOTAL_QUANTITY_SOLD, pa.CITY, sum( oi.quantity * oi.unit_Price) as REVENUE
from  product as p
join order_item as oi on oi.product_id=p.product_id
join order_header as oh on oh.ORDER_ID=oi.ORDER_ID
join order_contact_mech as ocm on ocm.ORDER_ID=oh.ORDER_ID
join contact_mech as cm on ocm.CONTACT_MECH_ID=cm.CONTACT_MECH_ID
join postal_address as pa on pa.CONTACT_MECH_ID=cm.CONTACT_MECH_ID
where pa.STATE_PROVINCE_GEO_ID = "NY" and oh.ORDER_TYPE_ID = "SALES_ORDER"
group by p.PRODUCT_ID, p.INTERNAL_NAME, pa.CITY
order by TOTAL_QUANTITY_SOLD desc
limit 10;

-- 4
select f.FACILITY_ID, f.FACILITY_NAME, COUNT(DISTINCT oh.ORDER_ID) AS TOTAL_ORDERS, SUM(oh.grand_total) AS TOTAL_REVENUE, '2023-01-01 to 2023-01-31' AS DATE_RANGE
from facility as f
join Inventory_Item as ii on ii.facility_id = f.facility_id
join product as p on p.product_id=ii.product_id
join order_item as oi on p.product_id = oi.product_ID
join order_header as oh on oh.ORDER_ID = oi.ORDER_ID
WHERE oh.ORDER_TYPE_ID = 'SALES_ORDER' AND oh.STATUS_ID = 'ORDER_COMPLETED'
AND oh.ORDER_DATE BETWEEN '2023-01-01' AND '2023-01-31 23:59:59'
group by f.FACILITY_ID, f.FACILITY_NAME 
order by f.facility_id desc;

-- 5
select distinct ii.INVENTORY_ITEM_ID, ii.PRODUCT_ID, ii.FACILITY_ID, iiv.quantity_On_Hand_Var as QUANTITY_LOST, iiv.variance_Reason_Id as REASON_CODE, ii.datetime_Received as TRANSACTION_DATE
from inventory_item as ii
join Inventory_Item_Variance as iiv on iiv.inventory_Item_Id=ii.inventory_Item_Id;

-- 6
select * from product_facility;

select p.PRODUCT_ID, p.PRODUCT_NAME, pf.FACILITY_ID, ii.quantity_On_Hand_Total as QOH, ii.available_To_Promise_Total as ATP, pf.minimum_stock as REORDER_THRESHOLD, 
NOW() as DATE_CHECKED
from product as p
join product_facility as pf on pf.product_id=p.product_id
join inventory_item as ii on ii.product_id=p.product_id and ii.facility_id=pf.facility_id
where ii.quantity_On_Hand_Total<= pf.minimum_stock OR ii.quantity_On_Hand_Total=0
order by pf.minimum_stock desc;

-- 7
select oh.ORDER_ID, oh.status_id as ORDER_STATUS, f.FACILITY_ID, f.FACILITY_NAME, f.FACILITY_TYPE_ID
from order_header as oh
join order_item_ship_group oisg on oh.order_id = oisg.order_id
join facility f on oisg.FACILITY_ID = f.FACILITY_ID
where oh.STATUS_ID in ("ORDER_CREATED" , "ORDER_APPROVED"); 

-- 8
select ii.PRODUCT_ID, ii.FACILITY_ID, ii.QUANTITY_ON_HAND_TOTAL as QOH, ii.AVAILABLE_TO_PROMISE_TOTAL as  ATP, 
(ii.quantity_on_hand_total - ii.available_to_promise_total) as DIFFERENCE
from inventory_item ii 
where ii.QUANTITY_ON_HAND_TOTAL != ii.AVAILABLE_TO_PROMISE_TOTAL;

-- 9
select * from order_status;

select os.ORDER_ID, os.ORDER_ITEM_SEQ_ID, os.status_id as CURRENT_STATUS_ID, os.status_datetime as STATUS_CHANGE_DATETIME, os.status_user_login as CHANGED_BY
from order_status as os;

-- 10
select oh.sales_Channel_Enum_Id as SALES_CHANNEL, count(oh.order_id) as TOTAL_ORDERS, sum(oh.grand_total) as TOTAL_REVENUE, DATE_FORMAT(oh.ORDER_DATE, '%Y-%m') AS REPORTING_PERIOD
from order_header as oh
group by SALES_CHANNEL, REPORTING_PERIOD;
