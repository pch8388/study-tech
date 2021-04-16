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

- 중복제거 참고용 => 이중으로 데이터 중복을 체크해야할 때 썼던 방법

```javascript
const _duplicateData = function(_data) {
    const houseDup = _data.reduce(function(p, c) {
        const _id = [c.houseCd, c.houseNm].join('|');

        if(p.temp.indexOf(_id) === -1) {
            p.out.push(c);
            p.temp.push(_id);
        }
        return p;
    }, {
        temp: [],
        out: []
    }).out;

    const room = _data.reduce(function(p, h) {
        return p.concat(h.rooms);
    }, []);

    const roomDup = room.reduce(function(p, c) {
        const _id = c.roomNm;
        if(p.temp.indexOf(_id) === -1) {
            p.out.push(c);
            p.temp.push(_id);
        }
        return p;
    }, {
        temp: [],
        out: []
    }).out;

    return _data.length !== houseDup.length || room.length !== roomDup.length;
};
```