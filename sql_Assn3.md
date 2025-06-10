``` sql
-- ASSIGNMENT 3-------------------------------------------

-- 1
select oh.ORDER_ID, oi.ORDER_ITEM_SEQ_ID, p.PRODUCT_ID, p.PRODUCT_TYPE_ID, oh.SALES_CHANNEL_ENUM_ID, oh.ORDER_DATE, oh.ENTRY_DATE, oh.STATUS_ID, os.STATUS_DATETIME,
 oh.ORDER_TYPE_ID, oh.PRODUCT_STORE_ID
from order_header oh 
join order_item oi on oh.ORDER_ID = oi.ORDER_ID
join order_status os on oh.ORDER_ID = os.ORDER_ID 
join product p on oi.PRODUCT_ID = p.PRODUCT_ID
join product_type  pt on p.PRODUCT_TYPE_ID = pt.PRODUCT_TYPE_ID
where oh.ORDER_TYPE_ID = "SALES_ORDER" and oh.STATUS_ID = "ORDER_COMPLETED" and pt.IS_PHYSICAL = "Y";

-- 2
select * from return_header;

select rh.RETURN_ID, oh.ORDER_ID, oh.PRODUCT_STORE_ID, rs.STATUS_DATETIME, oh.ORDER_NAME, rh.FROM_PARTY_ID, rh.RETURN_DATE, rh.ENTRY_DATE, rh.RETURN_CHANNEL_ENUM_ID
from return_header rh
join return_item ri on ri.RETURN_ID = rh.RETURN_ID AND ri.STATUS_ID = "RETURN_COMPLETED"
join order_header oh on ri.ORDER_ID = oh.ORDER_ID
join return_status rs on rh.RETURN_ID = rs.RETURN_ID AND rs.STATUS_ID = "RETURN_COMPLETED";

-- 3
select rh.from_party_id as PARTY_ID, per.FIRST_NAME
from return_header as rh
join person as per on per.party_id = rh.from_party_id
where rh.RETURN_DATE < DATE_FORMAT(NOW(), '%23-%m-01') and rh.RETURN_DATE >=  DATE_FORMAT(NOW() - INTERVAL 1 MONTH, '%23-%m-01') 
group by PARTY_ID, per.FIRST_NAME
having count(rh.RETURN_ID) = 1;

-- 4
select count(rh.RETURN_ID) as TOTAL_RETURNS, sum(ri.return_price * ri.return_quantity) as RETURN_TOTAL, count(ra.return_id)  as TOTAL_APPEASEMENTS, 
sum(ra.amount) as APPEASEMENTS_TOTAL 
from return_header rh 
join return_item ri on  rh.return_id = ri.return_id
join return_adjustment ra on ra.return_id  = rh.return_id AND ra.RETURN_ADJUSTMENT_TYPE_ID = "APPEASEMENT";

-- 5
select rh.RETURN_ID, rh.ENTRY_DATE, ra.RETURN_ADJUSTMENT_TYPE_ID, ra.AMOUNT, ra.COMMENTS, ri.ORDER_ID, oh.ORDER_DATE, rh.RETURN_DATE, oh.PRODUCT_STORE_ID
from return_header as rh
join RETURN_ADJUSTMENT as ra on rh.return_Id=ra.return_Id
join RETURN_ITEM as ri on ri.return_Id=rh.return_Id
join order_header as oh on oh.order_id= ri.order_id;

-- 6
select * from Return_Reason;
select ri.ORDER_ID, rh.RETURN_ID, rh.RETURN_DATE, rr.description as RETURN_REASON, ri.RETURN_QUANTITY
from return_header as rh
join return_item as ri on ri.return_id=rh.return_id
join Return_Reason as rr on rr.return_Reason_Id=ri.return_Reason_Id
where ri.ORDER_ID  IN ( select ORDER_ID  from return_item group by ORDER_ID having count(RETURN_ID) > 1 );

-- 7
select f.FACILITY_ID, f.FACILITY_NAME, count(oisg.order_id) as TOTAL_ONE_DAY_SHIP_ORDERS, 'Last one month' as REPORTING_PERIOD
from facility as f
join Order_Item_Ship_Group as oisg  on oisg.facility_id=f.facility_id AND oisg.SHIPMENT_METHOD_TYPE_ID = "NEXT_DAY"
join order_header as oh on oh.order_id = oisg.order_id
where oh.ORDER_DATE >= DATE_FORMAT(NOW() - INTERVAL 1 MONTH, '23-%m-01') 
AND oh.ORDER_DATE < DATE_FORMAT(NOW(), '23-%m-01') 
group by f.FACILITY_ID, f.FACILITY_NAME
order by TOTAL_ONE_DAY_SHIP_ORDERS desc
limit 1;

-- 8
select plr.PARTY_ID, per.first_name as NAME , plr.ROLE_TYPE_ID, pl.FACILITY_ID , p.status_id
from Picklist as pl
join Picklist_Role as plr on plr.picklist_Id= pl.picklist_Id
join party as p on p.party_id= plr.party_id
join person as per on per.party_id=p.party_id;

-- 9
select p.PRODUCT_ID, p.internal_Name as PRODUCT_NAME , count(pf.facility_Id)as FACILITY_COUNT ,  pf.FACILITY_ID
from product as p
join product_facility as pf on pf.PRODUCT_ID=p.PRODUCT_ID
group by p.PRODUCT_ID, PRODUCT_NAME, pf.FACILITY_ID;

-- 10
select ii.PRODUCT_ID, ii.FACILITY_ID, f.FACILITY_TYPE_ID, ii.quantity_On_Hand_Total as QOH, ii.available_To_Promise_Total as ATP
from Inventory_Item as ii
join facility as f on f.FACILITY_ID= ii.FACILITY_ID AND f.FACILITY_TYPE_ID != 'VIRTUAL_FACILITY';

-- 11
select * from inventory_transfer;
select it.inventory_transfer_id as TRANSFER_ORDER_ID, it.FACILITY_ID as FROM_FACILITY_ID, it.FACILITY_ID_TO , it.PRODUCT_ID, 
it.quantity as REQUESTED_QUANTITY, oisgir.QUANTITY RESERVED_QUANTITY, it.SEND_DATE as TRANSFER_DATE, it.status_id as STATUS
from inventory_transfer as it
join order_item_ship_grp_inv_res oisgir on oisgir.INVENTORY_ITEM_ID = it.INVENTORY_ITEM_ID;

-- 12 
select distinct
oh.order_id,
oh.order_date,
oh.status_id,
oisg.facility_id,
datediff(date(os.status_datetime), date(oh.entry_date)) as duration
from order_header oh
join order_item_ship_group oisg on oisg.ORDER_ID = oh.ORDER_ID
join order_status  os on os.ORDER_ID = oh.ORDER_ID
join picklist pl on pl.FACILITY_ID = oisg.FACILITY_ID
where pl.STATUS_ID is null

```
