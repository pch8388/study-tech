# 개요
- HTML5 에 포함된 스펙
- 쿠키의 단점 개선을 위해 도입

# 특징
- 쿠키는 매번 서버로 전송하지만 web storage 는 서버로 전송하지 않음
- web storage 는 객체정보 저장 가능
- 용량 제한 없음
    - 쿠키는 한 사이트당 20개, 최대 4kb
- 영구 저장 가능
- 도메인 단위 접근 제한
- localStorage : 명시적으로 삭제하지 않으면 영구 보관
- sessionStorage : 세션이 유지되는 동안 보관 (최대 5MB), 브라우저나 탭을 닫으면 사라짐(탭간 공유 안됨)
- StorageEvent : 새로운 항목을 추가하는 등 저장 공간이 변경될 때 발생 (Window.onstorage)