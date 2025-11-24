# Query-to-to-link-picking-tasks-generated-and-or-completed-in-Prod-to-the-transaction-name.
 To verify which pick transactions are triggered in Prod,  include INT50,51,55 and to join task_hdr or a similar table.

 
with latest_tasks as
(
    select  th.TASK_ID
           ,th.TASK_TYPE
           ,th.WHSE
           ,row_number() over ( partition by th.WHSE,th.TASK_TYPE order by  th.MOD_DATE_TIME desc ) as rn
    from TASK_HDR th
    where INVN_NEED_TYPE in (50, 55)
)
select  DISTINCT lt.TASK_ID as LATEST_TASK_ID
       ,ta.WHSE
       ,tm.TRAN_NAME
       ,ta.TASK_TYPE
       ,tm.TRAN_ID
       ,ta.INVN_NEED_TYPE
from TASK_ACTIon ta
join TRAN_MASTER tm
     on tm.TASK_NAME = ta.TASK_NAME
join latest_tasks lt
     on lt.TASK_TYPE = ta.TASK_TYPE and lt.WHSE = ta.WHSE
where tm.TRAN_TYPE in ('RF')
      and ta.MENU_MODE = 'I'
      and ta.WHSE in ( select WHSE from WHSE_SYS_CODE where WHSE not in ('01', 'ZZZ') group by WHSE )
      and lt.rn = 1;
