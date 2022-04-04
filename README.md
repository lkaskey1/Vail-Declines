# Vail-Declines

select
    bl.LOAN_ID as "Loan ID",
    bl.ACCOUNT_ID as "Account ID",
    bl.MERCHANT_COUNTRY as "Customer Location",
    bl.DISBURSEMENT_CREATED_TIMESTAMP as "Loan timestamp",
    bl.ORDER_AMOUNT as "Original order amount",
    bd.AMOUNT as "Auth requested",
    substring(bd.CARD_NUMBER,1,6) as "Card number - BIN",
    bd.RESPONSE as "Status",
    bd.DECLINE_REASON as "Card decline reason (from processor)",
    bd.MERCHANT_NAME as "Merchant name",
    bd_first.TIMESTAMP as "First authorization time (in UTC)",
    bd.TIMESTAMP as "Last authorization time (in UTC)",
    bd.AMOUNT - bl.ORDER_AMOUNT as "Difference in $",
    (bd.AMOUNT- bl.ORDER_AMOUNT) / bl.ORDER_AMOUNT as "Difference in %"
from BI_LOAN bl join BI_DISBURSEMENT bd on bl.APPLICATION_ID = bd.APPLICATION_ID join BI_DISBURSEMENT bd_first on bl.APPLICATION_ID = bd_first.APPLICATION_ID
where TRUE
and bl.APPLICATION_ID in (select bd2.APPLICATION_ID from BI_DISBURSEMENT bd2 where bd2.APPLICATION_ID=bl.APPLICATION_ID and bd2.OPERATION='AUTHORIZATION' and bd2.RESPONSE='Decline')
and bl.APPLICATION_ID not in (select bd3.APPLICATION_ID from BI_DISBURSEMENT bd3 where bd3.APPLICATION_ID=bl.APPLICATION_ID and bd3.OPERATION='AUTHORIZATION' and bd3.RESPONSE='Approval')
and bl.APPLICATION_ID not in (select bd4.APPLICATION_ID from BI_DISBURSEMENT bd4 where bd4.APPLICATION_ID=bl.APPLICATION_ID and bd4.OPERATION='SETTLEMENT')
and bd_first.PROCESSOR_TRANSACTION_ID in (select min(bd5.PROCESSOR_TRANSACTION_ID) from BI_DISBURSEMENT bd5 where bd5.APPLICATION_ID=bl.APPLICATION_ID and bd5.OPERATION='AUTHORIZATION' and bd5.RESPONSE='Decline' group by bd5.APPLICATION_ID)
and bd.PROCESSOR_TRANSACTION_ID in (select max(bd6.PROCESSOR_TRANSACTION_ID) from BI_DISBURSEMENT bd6 where bd6.APPLICATION_ID=bl.APPLICATION_ID and bd6.OPERATION='AUTHORIZATION' and bd6.RESPONSE='Decline' group by bd6.APPLICATION_ID)
and bl.UPCODE in ('UP-17367120-1','UP-17367120-2')
order by bl.LOAN_ID dsc
;
