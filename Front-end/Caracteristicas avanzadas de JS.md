## Definicion
una promesa (promise), es una accion que se ejecuta de manera asincrona que puede concluir en un timempo determinado. Puede generar un valor y notificar a los usuarios cuando concluya la tarea, la promeda piede resolverse de manera exitosa o puede fracasar.

## Ejemplo
### cuando se cumple correctamente

`<script>
	let promise = new Promise (function (resolve,reject){
		setTimeout(()=> resolve("terminado"), 1000)
	});
	//La funcion resolve se ejecutara con la instruccion .then
	promise.them(
		result=>alert(result),
		error=>alert(error)
`
### cuando no se cumple correctamente

`<script>
	let promise = new Promise (function (resolve,reject){
		setTimeout(()=> resolve("error inesperado"), 1000)
	});
	//La funcion resolve se ejecutara con la instruccion .then
	promise.them(
		result=>alert(result),
		error=>alert(error)
`