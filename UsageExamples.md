# Object.observe: API Usage Example

## Basic

```js
var records;
function observer(recs) {
  records = recs;
}

var obj = { id: 1 };
Object.observe(obj, observer);

obj.a = 'b';
obj.id++;
Object.defineProperty(obj, 'a', { enumerable: false });
delete obj.a;
Object.preventExtensions(obj);

Object.deliverChangeRecords(observer);
assertChangesAre(records, [
  { object: obj, type: 'add', name: 'a' },
  { object: obj, type: 'update', name: 'id', oldValue: 1 },
  { object: obj, type: 'reconfigure', name: 'a' },
  { object: obj, type: 'delete', name: 'a', oldValue: 'b' },
  { object: obj, type: 'preventExtensions', }
]);
```

## Array

```js
var basicChanges;
function basicObserver(recs) {
  basicChanges = recs;
}

var arrayChanges;
function arrayObserver(recs) {
  arrayChanges = recs;
}

var array = [1, 2, 3];
Object.observe(array, basicObserver);
Array.observe(array, arrayObserver);

array.push(4);
array.splice(2, 2);
array[5] = 'a';
array.length = 0;

Object.deliverChangeRecords(basicObserver);
assertChangesAre(basicChanges, [
  // push
  { object: array, type: 'add', name: '3' },
  { object: array, type: 'update', name: 'length', oldValue: 3 },
  // splice
  { object: array, type: 'delete', name: '3', oldValue: 4 },
  { object: array, type: 'delete', name: '2', oldValue: 3 },
  { object: array, type: 'update', name: 'length', oldValue: 4 },
  // index assignment
  { object: array, type: 'add', name: '5' },
  { object: array, type: 'update', name: 'length', oldValue: 2 },
  // length assignment
  { object: array, type: 'delete', name: '5', oldValue: 'a' },
  { object: array, type: 'delete', name: '1', oldValue: 2 },
  { object: array, type: 'delete', name: '0', oldValue: 1 },
  { object: array, type: 'update', name: 'length', oldValue: 6 },
]);

Object.deliverChangeRecords(arrayObserver);
assertChangesAre(arrayChanges, [
  // push
  { object: array, type: 'splice', index: 3, removed: [], addedCount: 1 },
  // splice
  { object: array, type: 'splice', index: 2, removed: [3, 4], addedCount: 0 },
  // index assignment
  { object: array, type: 'splice', index: 2, removed: [], addedCount: 4 },
  // length assignment
  { object: array, type: 'splice', index: 0, removed: [1, 2,,,,'a'], addedCount: 0 },
]);
```

## Synthetic Change Records

```js
function Circle(r) {
  var radius = r;

  var notifier = Object.getNotifier(this);
  function notifyAreaAndRadius(radius) {
    notifier.notify({
      type: 'update',
      name: 'radius',
      oldValue: radius
    })
    notifier.notify({
      type: 'update',
      name: 'area',
      oldValue: Math.pow(radius * Math.PI, 2)
    });
  }

  Object.defineProperty(this, 'radius', {
    get: function() {
      return radius;
    },
    set: function(r) {
      if (radius === r)
        return;
      notifyAreaAndRadius(radius);
      radius = r;
    }
  });

  Object.defineProperty(this, 'area', {
    get: function() {
      return Math.power(radius * Math.PI, 2);
    },
    set: function(a) {
      r = Math.sqrt(a)/Math.PI;
      notifyAreaAndRadius(radius);
      radius = r;
    }
  });
}

var changes;
function observer(recs) {
  changes = recs;
}

var circle = new Circle(5);

Object.observe(circle, observer);

circle.radius = 10;
circle.area = 100;

Object.deliverChangeRecords(observer);
assertChangesAre(changes, [
  // 'radius' assignment
  { object: circle, type: 'update', name: 'radius', oldValue: 5 },
  { object: circle, type: 'update', name: 'area', oldValue: 246.74011002723395 },
  // 'area' assignment
  { object: circle, type: 'update', name: 'radius', oldValue: 10 },
  { object: circle, type: 'update', name: 'area', oldValue: 986.9604401089358 },
]);
```



## Higher-level change records

```js
function Square(x, y, width, height) {
  this.x = x;
  this.y = y;
  this.width = width;
  this.height = height;
}

Square.prototype = {
  scale: function(ratio) {
    Object.getNotifier(this).performChange('scale', () => {
      this.width *= ratio;
      this.height *= ratio;
      return {
        ratio: ratio
      };
    });
  },

  translate: function(dx, dy) {
    Object.getNotifier(this).performChange('translate', () => {
      this.x += dx;
      this.y += dy;
      return {
        dx: dx,
        dy: dy
      }
    });
  }
}

Square.observe = function(square, callback) {
  return Object.observe(square, callback, ['update', 'translate', 'scale']);
}

var basicChanges;
function basicObserver(recs) {
  basicChanges = recs;
}

var squareChanges;
function squareObserver(recs) {
  squareChanges = recs;
}

var square = new Square(0, 0, 10, 10);

Object.observe(square, basicObserver);
Square.observe(square, squareObserver);

square.translate(5, 5);
square.x = -5;
square.scale(2);

Object.deliverChangeRecords(basicObserver);
assertChangesAre(basicChanges, [
  // translate
  { object: square, type: 'update', name: 'x', oldValue: 0 },
  { object: square, type: 'update', name: 'y', oldValue: 0 },
  // assignment to 'x'
  { object: square, type: 'update', name: 'x', oldValue: 5 },
  // scale
  { object: square, type: 'update', name: 'width', oldValue: 10 },
  { object: square, type: 'update', name: 'height', oldValue: 10 },
]);

Object.deliverChangeRecords(squareObserver);
assertChangesAre(squareChanges, [
  // translate
  { object: square, type: 'translate', dx: 5, dy: 5 },
  // assignment to 'x'
  { object: square, type: 'update', name: 'x', oldValue: 5 },
  // scale
  { object: square, type: 'scale', ratio: 2 },
]);
```
