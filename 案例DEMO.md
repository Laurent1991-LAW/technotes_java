案例DEMO





### 请假审批系统设计

**接口 :**  

calculateDayGap(Date start, Date end) ;

submitRequest(Request request) ; ---- 幂等性设计



**表1 : 申请表**

- id, name, num, phone,
- startDate, endDate,
- reason, 
- status (-2 废除, -1 驳回, 0 审批中, 1 通过)



**表2 : 流程表**

- id, require_id,
- count,
- stage,
- fir_handler, sec_handler, thir_handler
- cur_handler



### 审批小组成员存取



