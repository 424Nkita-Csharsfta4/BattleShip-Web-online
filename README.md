# BattleShip-Web-online

# Морской Бой
Node.js, Express, Socket.io

Получилось блогодаря этой информации и видео 
```
http://cleanjs.ru/articles/igra-morskoj-boj-na-chistom-javascript-rasstanovka-korablej.html
```
`````
https://proglib.io/p/rasprostranennye-algoritmy-i-struktury-dannyh-v-javascript-grafy-2021-10-20
`````
![image](https://user-images.githubusercontent.com/103760832/201512922-152fc0c6-23db-41e7-91be-a2ceb007fbbe.png)

![image](https://user-images.githubusercontent.com/103760832/201512942-d28cdc47-fe9a-40c7-9777-b3b5883e7ccf.png)

![image](https://user-images.githubusercontent.com/103760832/201513011-22efc34b-ee68-47d0-85cd-ccd585ea4aee.png)

![image](https://user-images.githubusercontent.com/103760832/201513016-ab762cb9-ae60-45cc-bcf8-7ef3512b1858.png)

```
Серер код Socket.io + Node.js
````
![image](https://user-images.githubusercontent.com/103760832/201513118-186ba48a-374e-46fc-be88-bbad591932c8.png)

````
Игровая логика для онлайна
````

![image](https://user-images.githubusercontent.com/103760832/201513219-64d73139-38f6-4fd5-8a36-621d6deb99de.png)

````
```
jqury draggable touch библиотека переноса - 
https://jqueryui.com/draggable/

```
```
jQuery UI Shake Effect - 
https://www.geeksforgeeks.org/jquery-ui-shake-effect/
```

````
[Документация]
https://k6.io/docs/javascript-api/k6-ws/socket/socket-on/
````
[Создание комнаты]
app.get('/create', async function (request, response){
    let roomId = await createRoom()
    console.log(`Комната создана: ${roomId}`)
    response.redirect(`/room/${roomId}`)
})
app.get('/create/:idLength', async function (request, response){
    let idLength = parseInt(request.params.idLength) || ID_LENGTH 
	let roomId = await createRoom((idLength <= 20) ? idLength : ID_LENGTH)
    console.log(`Комната создана: ${roomId} с длиной ID ${idLength}`)
	response.redirect(`/room/${roomId}`)
})
````
````
[Нахождение комнаты]
app.get('/room/:roomId', async function(request, response) {
	let roomId = request.params.roomId
    let currentRoom = await getRoomById(roomId)
	if (currentRoom){
		console.log(`Комната найдена: ${roomId}`)
		response.render(__dirname + '/static/html/index.html', {roomId: roomId});
	}
	else {
        response.redirect('/')
    }
})
````

````
[Корабли]
class Ship {
	constructor(index, size, DOMElement){
		this.index = index
		this.size = size
		this.element = DOMElement
		this.orientation = 'h'
		this.segments = new Array(size)
	}
}
````
````
[Генирацмя кораблей]
function generateRandomField(){//сгенерировать случайное поле
	let field = []
	let shipsSizes = [4, 3, 3, 2, 2, 2, 1, 1, 1, 1]

	for (let index = 0; index < shipsSizes.length; index++){
		let size = shipsSizes[index]
		let creatingShip = true
		while(creatingShip){
			console.log(creatingShip)
			let newSegments = new Array(size)
			let orientation = (size !== 1) ? ['h', 'v'][randInt(0, 1)] : 'h'
			let x = randInt(1, (orientation === 'h') ? 10 - size + 1: 10)
			let y = randInt(1, (orientation === 'v') ? 10 - size + 1: 10)
			if (orientation === 'h'){
				for (let i = 0; i < size; i++){
					newSegments[i] = {x: x + i, y: y}
				}
			}
			else if (orientation === 'v'){
				for (let i = 0; i < size; i++){
					newSegments[i] = {x: x, y: y + i}
				}
			}

			let canBePlaced = true

			for (let thisSegment of newSegments){
				for (let ship of field){
					for (let segment of ship.segments){
							for (let offset of offsets){
							if (segment.x + offset.x === thisSegment.x && segment.y + offset.y === thisSegment.y) {
								canBePlaced = false
							}
						}
					}
				}
			}

			if (canBePlaced){
				let newShip = new Ship(index, size, ships[index])
				newShip.segments = newSegments
				newShip.orientation = orientation
				field.push(newShip)
				creatingShip = false
			}
		}
	}
	return field
}
````
````
[Повернуть корабли]
function rotateShip(index){
	let thisShip = field[index]
	let newSegments = new Array(thisShip.size)

	if (thisShip.element.classList.contains('dragging') || thisShip.size === 1 || thisShip.segments.every(segment => segment == undefined)) return 

	if ((function(){
		if (thisShip.orientation === 'h'){
			for (let i = 0; i < thisShip.size; i++){
				newSegments[i] = {x: thisShip.segments[0].x, y: thisShip.segments[0].y + i}
			}
		}
		else if (thisShip.orientation === 'v'){
			for (let i = 0; i < thisShip.size; i++){
				newSegments[i] = {x: thisShip.segments[0].x + i, y: thisShip.segments[0].y}
			}
		}
		for (let thisSegment of newSegments){
			if (!(thisSegment.x >= 1 && thisSegment.x <= 10 && thisSegment.y >= 1 && thisSegment.y <= 10)) return false
			for (let ship of field){
				if (ship === thisShip) continue
				for (let segment of ship.segments){
						if (!segment) continue
						for (let offset of offsets){
						if (segment.x + offset.x === thisSegment.x && segment.y + offset.y === thisSegment.y) {
							return false
						}
					}
				}
			}
		}
		return true
	})()){
		thisShip.segments = newSegments
		if (thisShip.orientation === 'h') {
			thisShip.orientation = 'v'
			thisShip.element.removeClassIfContains('horizontal')
			thisShip.element.addClassIfNotContains('vertical')
		}
		else if (thisShip.orientation === 'v') {
			thisShip.orientation = 'h'
			thisShip.element.removeClassIfContains('vertical')
			thisShip.element.addClassIfNotContains('horizontal')
		}
	}
	else {
		shakeShip(index)
	}
}
````
````
[Перевернуть корвбли]
function revertShip(index){
	let thisShip = field[index]

	thisShip.segments = new Array(thisShip.size)
	thisShip.orientation = 'h'

	thisShip.element.addClassIfNotContains('automove')
	thisShip.element.removeClassIfContains('vertical')
	thisShip.element.addClassIfNotContains('horizontal')

	thisShip.element.style.top  = '0px'
	thisShip.element.style.left = '0px'

	setTimeout(function(){
		this.removeClassIfContains('automove')
	}.bind(thisShip.element), 350)

	validateField()
}

function shakeShip(index){$(`.ship:eq(${index})`).effect('shake', {times: 2, distance: 3}, 300, function(){this.removeClassIfContains('shake')}).addClass('shake')}

function moveShip(index, x, y, withAnimation = false){
	let thisShip = field[index]
	let gridRect = query('.battlefield.player').getBoundingClientRect()
	let parentRect = thisShip.element.parentElement.getBoundingClientRect()

	if (withAnimation) thisShip.element.addClassIfNotContains('automove')

	thisShip.element.style.top  = gridRect.top  + y * cellSize - parentRect.top  + 'px'
	thisShip.element.style.left = gridRect.left + x * cellSize - parentRect.left + 'px'

	if (withAnimation) setTimeout(function(){
		this.removeClassIfContains('automove')
	}.bind(thisShip.element), 350)
}
````
Запуск игры
````
```
npm install

node server.js
```




