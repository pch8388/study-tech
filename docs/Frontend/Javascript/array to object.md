- 첫째 배열 요소의 정보로 두번째 배열의 요소들을 찾아서 매핑 시켜줌
- 최종 데이터 셋은 ```{main: [subvalues]}```
```javascript
const categories = [...view.environmentLayout.querySelectorAll('.jsEnvMainCheck:checked')]
  .reduce((_obj, el) => {
      const divEl = el.closest('.checkbox_info_cover');
      const subValues = [...divEl.querySelectorAll('.jsEnvSubCheck:checked')].map(sub => sub.value);

	// {이전값을 구조분해할당, [key]: value}해서 object로 바꿈
  return {..._obj, [el.value]: subValues};
  }, {});
```