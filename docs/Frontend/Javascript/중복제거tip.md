# array 중복 제거

- 여러 필드를 합쳐서 중복제거를 해야할 때 사용할 수 있다

```javascirpt
return arr.reduce(function(p, c) {
	const _id = [c.seq, c.name].join('|');

	if(p.temp.indexOf(_id) === -1) {
		p.out.push(c);
		p.temp.push(_id);
	}
	return p;
}, {
	temp: [],
	out: []
}).out;
```