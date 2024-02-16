# DB-.sql
-------------------------------------------------------------------------------------------------------------------------------------
-- 시스템 예외 
-------------------------------------------------------------------------------------------------------------------------------------
-- divisor is equal to zero 오류 
DECLARE 
   vi_num NUMBER := 0;
BEGIN
	 vi_num := 10 / 0;
	 DBMS_OUTPUT.PUT_LINE('Success!');     --- 실행되지 않음
EXCEPTION WHEN OTHERS THEN
	 DBMS_OUTPUT.PUT_LINE('오류가 발생했습니다');	
END;

-----------------------------------------------------  오류구문 
CREATE OR REPLACE PROCEDURE ch10_no_exception_proc 
IS
  vi_num NUMBER := 0;
BEGIN
    vi_num := 10 / 0;
    DBMS_OUTPUT.PUT_LINE('Success!');
END;	
------------------------------------------------------ 오류가 나지 않게 수정
CREATE OR REPLACE PROCEDURE ch10_exception_proc 
IS
  vi_num NUMBER := 0;
BEGIN
    vi_num := 10 / 0;
    DBMS_OUTPUT.PUT_LINE('Success!');
EXCEPTION WHEN OTHERS THEN
     DBMS_OUTPUT.PUT_LINE('오류가 발생했습니다');		
END;	

BEGIN
     ch10_no_exception_proc;            -- 해당 프로시져 오류 발생으로 중단됨
     DBMS_OUTPUT.PUT_LINE('Success!');
END;
 ---------------------------------------------
BEGIN
     ch10_exception_proc;               -- 해당 프로시져 오류 발생되지만 예외처리로 
     DBMS_OUTPUT.PUT_LINE('Success!');  -- 중단되지 않고 정상종료 
END;


-- SQLCODE 실행부에서 발생한 예외에 해당하는 코드를 반환
-- SQLERRM 발생한 예외에 대한 오류 메시지 반환.

CREATE OR REPLACE PROCEDURE ch10_exception_proc 
IS
  vi_num NUMBER := 0;
BEGIN
    vi_num := 10 / 0; 
    DBMS_OUTPUT.PUT_LINE('Success!');
    
EXCEPTION WHEN OTHERS THEN
 DBMS_OUTPUT.PUT_LINE('오류가 발생했습니다');		
 DBMS_OUTPUT.PUT_LINE( 'SQL ERROR CODE: '    || SQLCODE);
 DBMS_OUTPUT.PUT_LINE( 'SQL ERROR MESSAGE: ' || SQLERRM); -- 매개변수 없는 SQLERRM
END;	

EXEC ch10_exception_proc;


-- 시스템 예외명 있는 오류 

CREATE OR REPLACE PROCEDURE ch10_exception_proc 
IS
  vi_num NUMBER := 0;
BEGIN
	vi_num := 10 / 0;
	DBMS_OUTPUT.PUT_LINE('Success!');
EXCEPTION WHEN ZERO_DIVIDE THEN      --<-- ZERO_DIVIDE 정의되어있는 오류 
	 DBMS_OUTPUT.PUT_LINE('오류가 발생했습니다');		
	 DBMS_OUTPUT.PUT_LINE('SQL ERROR CODE: ' || SQLCODE);
	 DBMS_OUTPUT.PUT_LINE('SQL ERROR MESSAGE: ' || SQLERRM);
END;	

EXEC ch10_exception_proc;

CREATE OR REPLACE PROCEDURE ch10_exception_proc 
IS
  vi_num NUMBER := 0;
BEGIN
	vi_num := 10 / 0;
	DBMS_OUTPUT.PUT_LINE('Success!');
	
EXCEPTION WHEN ZERO_DIVIDE THEN
	          	 DBMS_OUTPUT.PUT_LINE('오류1');		
	             DBMS_OUTPUT.PUT_LINE('SQL ERROR MESSAGE1: ' || SQLERRM);
	        WHEN OTHERS THEN
	          	 DBMS_OUTPUT.PUT_LINE('오류2');		
	             DBMS_OUTPUT.PUT_LINE('SQL ERROR MESSAGE2: ' || SQLERRM);	
END;	

EXEC ch10_exception_proc;


CREATE OR REPLACE PROCEDURE ch10_upd_jobid_proc 
                  ( p_employee_id employees.employee_id%TYPE,
                    p_job_id      jobs.job_id%TYPE )
IS
  vn_cnt NUMBER := 0;
BEGIN
	SELECT COUNT(*)
	  INTO vn_cnt
	  FROM JOBS
	 WHERE JOB_ID = p_job_id;
	IF vn_cnt = 0 THEN
	   DBMS_OUTPUT.PUT_LINE('job_id가 없습니다');
	   RETURN;
	ELSE
	   UPDATE employees
	      SET job_id = p_job_id
	    WHERE employee_id = p_employee_id;
  END IF;
  
  COMMIT;
	
END;

EXEC ch10_upd_jobid_proc (200, 'SM_JOB2');



CREATE OR REPLACE PROCEDURE ch10_upd_jobid_proc 
                  ( p_employee_id employees.employee_id%TYPE,
                    p_job_id      jobs.job_id%TYPE )
IS
  vn_cnt NUMBER := 0;
BEGIN
	SELECT 1
	  INTO vn_cnt
	  FROM JOBS
	WHERE JOB_ID = p_job_id;
	 
   UPDATE employees
      SET job_id = p_job_id
    WHERE employee_id = p_employee_id;
	
  COMMIT;
  
  EXCEPTION WHEN NO_DATA_FOUND THEN
                 DBMS_OUTPUT.PUT_LINE(SQLERRM);
                 DBMS_OUTPUT.PUT_LINE(p_job_id ||'에 해당하는 job_id가 없습니다');
            WHEN OTHERS THEN
                 DBMS_OUTPUT.PUT_LINE('기타 에러: ' || SQLERRM);
END;
                   

EXEC ch10_upd_jobid_proc (100, 'SM_JOB4');


CREATE OR REPLACE PROCEDURE ch10_upd_jobid_proc 
                  ( p_employee_id employees.employee_id%TYPE,
                    p_job_id      jobs.job_id%TYPE)
IS
  vn_cnt NUMBER := 0;
BEGIN
	
	SELECT 1
	  INTO vn_cnt
	  FROM JOBS
	 WHERE JOB_ID = p_job_id;
	 
   UPDATE employees
      SET job_id = p_job_id
    WHERE employee_id = p_employee_id;
	
  COMMIT;
  
  EXCEPTION WHEN NO_DATA_FOUND THEN
                 DBMS_OUTPUT.PUT_LINE(SQLERRM);
                 DBMS_OUTPUT.PUT_LINE(p_job_id ||'에 해당하는 job_id가 없습니다');
            WHEN OTHERS THEN
                 DBMS_OUTPUT.PUT_LINE('기타 에러: ' || SQLERRM);
END;



/*
-------------------------------------------------------------------------------------------------------------------------------------
 1.사용자 정의 예외
-------------------------------------------------------------------------------------------------------------------------------------
   시스템 예외 이외에 사용자가 직접 예외를 정의 
   개발자가 직접 예외를 정의하는 방법.
-------------------------------------------------------------------------------------------------------------------------------------
[1] 사용자 예외 정의방법 
 (1) 예외 정의 : 사용자_정의_예외명 EXCEPTION;
 (2) 예외발생시키기 : RAISE 사용자_정의_예외명;
        시스템 예외는 해당 예외가 자동으로 검출 되지만, 
        사용자 정의 예외는 직접 예외를 발생시켜야 한다.
        RAISE 예외명 형태로 사용한다.
 (3) 발생된 예외 처리 : EXCEPTION WHEN 사용자_정의_예외명 THEN ..
*/
    CREATE OR REPLACE PROCEDURE ch10_ins_emp_proc ( 
                      p_emp_name       employees.emp_name%TYPE,
                      p_department_id  departments.department_id%TYPE )
    IS
       vn_employee_id  employees.employee_id%TYPE;
       vd_curr_date    DATE := SYSDATE;
       vn_cnt          NUMBER := 0;
       ex_invalid_depid EXCEPTION; -- (1) 잘못된 부서번호일 경우 예외 정의
    BEGIN
	     -- 부서테이블에서 해당 부서번호 존재유무 체크
	     SELECT COUNT(*)
	       INTO vn_cnt
	       FROM departments
	      WHERE department_id = p_department_id;
	     IF vn_cnt = 0 THEN
	        RAISE ex_invalid_depid; -- (2) 사용자 정의 예외 발생
	     END IF;
	     -- employee_id의 max 값에 +1
	     SELECT MAX(employee_id) + 1
	       INTO vn_employee_id
	       FROM employees; 
	     -- 사용자예외처리 예제이므로 사원 테이블에 최소한 데이터만 입력함
	     INSERT INTO employees ( employee_id, emp_name, hire_date, department_id )
                  VALUES ( vn_employee_id, p_emp_name, vd_curr_date, p_department_id );
       COMMIT;        
          
    EXCEPTION WHEN ex_invalid_depid THEN --(3) 사용자 정의 예외 처리구간 
                   DBMS_OUTPUT.PUT_LINE('해당 부서번호가 없습니다');
              WHEN OTHERS THEN
                   DBMS_OUTPUT.PUT_LINE(SQLERRM);              
    END;                	

EXEC ch10_ins_emp_proc ('홍길동', 10);

SELECT *
FROM employees
WHERE emp_name like '홍길동'; 


/*
--[2]시스템 예외에 이름 부여하기----------------------------------------------------------------------------------------------
 시스템 예외에는 ZERO_DIVIDE, INVALID_NUMBER .... 와같이 정의된 예외가 있다 하지만 이들처럼 예외명이 부여된 것은 
 시스템 예외 중 극소수이고 나머지는 예외코드만 존재한다. 이름이 없는 코드에 이름 부여하기.

	1.사용자 정의 예외 선언 
	2.사용자 정의 예외명과 시스템 예외 코드 연결 (PRAGMA EXCEPTION_INIT(사용자 정의 예외명, 시스템_예외_코드)

		/*
		   PRAGMA 컴파일러가 실행되기 전에 처리하는 전처리기 역할 
		   PRAGMA EXCEPTION_INIT(예외명, 예외번호)
		   사용자 정의 예외 처리를 할 때 사용되는것으로 
		   특정 예외번호를 명시해서 컴파일러에 이 예외를 사용한다는 것을 알리는 역할 
		   (해당 예외번호에 해당되는 시스템 에러시 발생) 
		*/
	3.발생된 예외 처리:EXCEPTION WHEN 사용자 정의 예외명 THEN ....
*/

CREATE OR REPLACE PROCEDURE ch10_ins_emp_proc ( 
                  p_emp_name       employees.emp_name%TYPE,
                  p_department_id  departments.department_id%TYPE,
                  p_hire_month  VARCHAR2  )
IS
   vn_employee_id  employees.employee_id%TYPE;
   vd_curr_date    DATE := SYSDATE;
   vn_cnt          NUMBER := 0;
   ex_invalid_depid EXCEPTION; -- 잘못된 부서번호일 경우 예외 정의
   
   ex_invalid_month EXCEPTION; -- 잘못된 입사월인 경우 예외 정의
   PRAGMA EXCEPTION_INIT (ex_invalid_month, -1843); -- 예외명과 예외코드 연결
BEGIN
	 -- 부서테이블에서 해당 부서번호 존재유무 체크
	 SELECT COUNT(*)
	   INTO vn_cnt
	   FROM departments
	 WHERE department_id = p_department_id;
	 IF vn_cnt = 0 THEN
	    RAISE ex_invalid_depid; -- 사용자 정의 예외 발생
	 END IF;
	 -- 입사월 체크 (1~12월 범위를 벗어났는지 체크)
	 IF SUBSTR(p_hire_month, 5, 2) NOT BETWEEN '01' AND '12' THEN
	    RAISE ex_invalid_month; -- 사용자 정의 예외 발생
	 END IF;
	 -- employee_id의 max 값에 +1
	 SELECT MAX(employee_id) + 1
	   INTO vn_employee_id
	   FROM employees;
	 -- 사용자예외처리 예제이므로 사원 테이블에 최소한 데이터만 입력함
	 INSERT INTO employees ( employee_id, emp_name, hire_date, department_id )
              VALUES ( vn_employee_id, p_emp_name, TO_DATE(p_hire_month || '01'), p_department_id );
   COMMIT;              
EXCEPTION WHEN ex_invalid_depid THEN -- 사용자 정의 예외 처리
               DBMS_OUTPUT.PUT_LINE('해당 부서번호가 없습니다');
          WHEN ex_invalid_month THEN -- 입사월 사용자 정의 예외
               DBMS_OUTPUT.PUT_LINE(SQLCODE);
               DBMS_OUTPUT.PUT_LINE(SQLERRM);
               DBMS_OUTPUT.PUT_LINE('1~12월 범위를 벗어난 월입니다');               
          WHEN OTHERS THEN
               DBMS_OUTPUT.PUT_LINE(SQLERRM);              	
END;    
EXEC ch10_ins_emp_proc ('홍길동', 110, '201314');
/*
 [3].사용자 예외를 시스템 예외에 정의된 예외명을 사용----------------------------------------------------------------------------------------------
      RAISE 사용자 정의 예외 발생시 
      오라클에서 정의 되어 있는 예외를 발생 시킬수 있다. 
*/
CREATE OR REPLACE PROCEDURE ch10_raise_test_proc ( p_num NUMBER)
IS
BEGIN
	IF p_num <= 0 THEN
	   RAISE INVALID_NUMBER;  -- 해당 조건일때 다른처리를 할때
  END IF;
  DBMS_OUTPUT.PUT_LINE(p_num);
EXCEPTION WHEN INVALID_NUMBER THEN
               DBMS_OUTPUT.PUT_LINE('양수만 입력받을 수 있습니다');
          WHEN OTHERS THEN
               DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;
EXEC ch10_raise_test_proc (-10);   

/*
[4].예외를 발생시킬 수 있는 내장 프로시저 ----------------------------------------------------------------------------------------------
  RAISE_APPLICATOIN_ERROR(예외코드, 예외 메세지);
  예외 코드와 메세지를 사용자가 직접 정의  -20000 ~ -20999 번까지 만 사용가능 
   왜냐면 오라클에서 이미 사용하고 있는 예외들이 위 번호 구간은 사용하지 않고 있기 때문에)
*/
CREATE OR REPLACE PROCEDURE ch10_raise_test_proc ( p_num NUMBER)
IS
BEGIN
	IF p_num <= 0 THEN
	   RAISE_APPLICATION_ERROR (-20000, '양수만 입력받을 수 있단 말입니다!');
	END IF;  
  DBMS_OUTPUT.PUT_LINE(p_num);
EXCEPTION WHEN OTHERS THEN
               DBMS_OUTPUT.PUT_LINE(SQLCODE);
               DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;
EXEC ch10_raise_test_proc (-10);               







-- (1) DML문이란(조작어)
--      SQL_DML : SELECT, INSERT, UPDATE, DELETE
--                SELECT : 실행순서 FROM ->WHERE->GROUP BY-HAVING ->SELECT->ORDER BY
-- (2) 조인이란?
--     INNER JOIN 과 OUTER JOIN 차이점.
-- (3) WHERE 검색조건과 HAVING 조건의 차이점
--     HAVING은 집계된 데이터에 대한 조건으로 검색
-- (4) 기본키란? PK 
-- (5) 서브퀄  -- select절, from절, where절
