# 목적
일반적인 java.lang.Exception Hooking 코드로 반환된 Java method call stack 값의 argument를 추적하기 위해 개발 및 연구 진행

# 사용법
기본적인 동작방식은
Target setup -> Script Setup -> Run Script 입니다.
현재 활성화된 타겟은 메뉴 아래에 세션(pid)와 클래스명으로 확인 가능하며,
현재 삽입된 JScode는 2. Show data -> 1. Show current JS code에서 확인 할 수 있습니다.

# 사용 가능한 스크립트
1. Error overloading : 타겟 함수의 오버로딩 값을 얻기 위해 실행하는 스크립트
2. Trace exception : 얻은 Overloading 값을 통해 Exception 발생시켜  Method에 대한 Call Stack을 얻기 위해 사용하는 스크립트
3. Trage argument of exception : 2번 스크립트에서 얻은 Call Stack을 활용하여 Call Stack안 메소드들의 Argument 값을 추적하기 위해 사용하는 스크립트

# Exception의 Argument 추적을 하기위한 스크립트 사용법
3. Setup to Script의 메뉴 1.->2.->3.을 순차적으로 Setup하여 실행하면 Argument 추적이 가능합니다.
각각의 스크립트 실행전에 결과 반환->결과 활용 인과관계에 대해서 설명하면,
1번 스크립트의 결과로 Overloading 내역이 런타임 이후 반환되며 반환된 값 중 추적하려는 값을 셋업하면
2번 스크립트의 Overloading 값으로 2번의 script.on 실행 전, 삽입되는 JScode 상자동으로 들어갑니다.
2번 스크립트 실행 이후 실행한 스크립트가 후킹되어 동작하면서 확인된 메소드 인스턴스들을 리스트 형태로 반환합니다.
이 리스트안에 담긴 객체는 각자 런타임에 가졌던 Exception Call Stack 정보를 가지고 있으며, 이를 3번 스크립트에서
활용하기 위해 1. Setup -> 2. Setup to data that haveto trace in exception result 메뉴에서 원하는 
Exception 인스턴스를 선택하여 3번 스크립트 실행전에 셋업합니다.
이후 3번 스크립트를 실행합니다.

TODO :
3번 스크립트 특정 스레드에 대해서만 추적하는 기능 필요
Exception Call Stack의 argument에 대한 평문 처리 필요
다른 앱들에서 활용 가능한 범용성 확장
iOS에서 활용 가능하도록 코드 구현 필요


# 코드 설명 주석 부분
프로그램 시작 495 : if __name__ == "__main__":
프로그램 종료 520 : while(1)의 break;

함수
  program_log : 프로그램 전역 로깅 함수

클래스
  Menu : 메인 메뉴 상호작용 및 출력
    함수
      show_menu : 메뉴 출력
      choice_check : 사용자 메뉴 선택값 입력 처리

  ClassFrida : Frida 후킹 클래스
    클래스
      ClassExceptionData
    함수 : 기능별 분류
      기본
      run_script : Frida 스크립트 실행
      select_script : 실행할 스크립트 선택
      show_selected_exception : show_exception_data에서 선택한 exception의 call stack을 출력
      show_script : 현재 설정된 스크립트를 출력
      show_exception_data : Exception 정보를 출력
      setup_is_init : Setup하는 Target의 메소드가 생성자인지 확인

      submenu_show_data : Show_data의 서브 메뉴
      submenu_setup_frida : Setup의 서브 메뉴

      설정 셋업
      setup_frida : Target을 셋업하는 함수
      setup_exception_data : Trace할 Exception을 선택
      setup_selected_exception_data : 2번 jscode를 실행 후 만들어진 exception 정보를 trace하기 위해 setup하는 함수

      JScode 생성
      setup_exception_trace_code : Exception을 Implementation한 jscode 생성
      setup_err_js_code : Overloading 값을 출력하기 위한 Error 발생 jscode jscode 생성
      create_tracing_exception_args_jscode : argument trace jscode 생성 함수
      template_jscode : Argument Trace jscode를 생성하는 클로저 반환 함수
      template_check_object_jscode : argument값이 object인지 확인하는 jscode 삽입 함수
      error_wrapper_setup_jscode : setup_err_js_code에서 생성된 jscode를 target에 삽입하는 함수

      검사
      check_exception_run_count : exception을 저장하기위한 g_count 검사
      check_send : exception 배열을 다음 인덱스로 증가시키기 위한 arguemnt 갯수 확인
      check_setup_script : script가 비어있는지 검사
      check_setup_target : target이 비어있는지 검사

      핸들 == 모니터링 // frida.script.on
      monitor_error : 1번 error jscode의 모니터링 함수
      monitor_exception : 2번 exception implementation jscode를 모니터링하는 함수
      monitor_exception_trace : 3번 argument trace jscode 모니터링 함수

      실행 후 처리
      parse_from_parameter : exception jscode에 삽입될 파라미터를 overloading 갯수를 측정하여 param1 , param2 의 형식으로 만들어주는
      parse_selected_exception_data : exception의 callstack을 파싱하는 함수
