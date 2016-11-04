## 使用$AngularJS$开发的注意事项和最佳实践



**解决双大括号绑定元素时的闪烁问题**

在$AngularJS$中，要等到DOM元素加载完成后，才会对页面进行解析和渲染。

在操作之前，应先对双大括号"{{}}"绑定的元素进行隐藏，可以向元素中添加"ng-cloak"

属性实现隐藏的效果；若绑定纯文字内容 ，使用"ng-bind"方式代替"{{}}"方式绑定数据（一般在首次加载的时候这样做）。

```html
        <div id="template" ng-cloak>{{message}}</div>
```



---



**使用ng-repeat时的注意事项**

在使用ng-repeat时，如果有过滤器时，调用$index不能准确定位到对应的记录，必须使用实际item对象。





---



**使用track by排序ng-repeat中的数据**

在使用ng-repeat指令显示数据列表时，如果需要更新数据 ，那么页面中原有的DOM元素在更新过程中并不会被重用，而是会被删除，再重新生成与上一次结构一样的元素。反复生成DOM会延迟加载速度，浪费资源。可以用track by对数据源进行排序。

```html
        <ul ng-repeat="user in users track by user.id">
            <li>
                <span>{{user.id}}</span>
                <span>{{user.name}}</span>
            </li>
        </ul>
```

[使用track by排序ng-repeat中的数据](https://github.com/aaronzhou0101/Notes/blob/master/slnAngular/Ch10/10-5%20%E4%BD%BF%E7%94%A8track%20by%E6%8E%92%E5%BA%8Fng-repeat%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE.html)

---



**理解ng-repeat指令中scope的继承关系**

ng-repeat在新建DOM时，也为每个新建的DOM元素创建了独立的scope作用域。他们有同一个父级作用域（控制器注入的$scope对象），调用`angular.element(domElement).scope`可以获取某个DOM元素所对应的作用域，再访问其父级作用域，修改绑定的数据源。

```javascript
            angular.module('a10_6', [])
            .controller('c10_6', function ($scope) {
                $scope.users = [{
                    id: 1010, name: "张立秋", score: 10
                }, {
                    id: 1020, name: "李山浃", score: 50
                }, {
                    id: 1030, name: "胡正清", score: 70
                }];
                $scope.change1 = function () {
                    var scope1 = angular.element(document.getElementById("spn1010")).scope();
                    var scope2 = angular.element(document.getElementById("spn1020")).scope();
                    console.log(scope1 == scope2);
                };
                $scope.change2 = function () {
                    var scope = angular.element(document.getElementById("spn1020")).scope();
                    console.log(scope.$parent == $scope);
                };
                $scope.change3 = function () {
                    var scope = angular.element(document.getElementById("spn1030")).scope();
                    scope.$parent.users = [{
                        id: 1040, name: "刘三夫", score: 40
                    }, {
                        id: 1050, name: "闻钟华", score: 50
                    }, {
                        id: 1060, name: "钱少忠", score: 60
                    }];
                };
            });
```



[理解ng-repeat指令中scope的继承关系](https://github.com/aaronzhou0101/Notes/blob/master/slnAngular/Ch10/10-6%20%E6%AD%A3%E7%A1%AE%E7%90%86%E8%A7%A3ng-repeat%E6%8C%87%E4%BB%A4%E4%B8%ADscope%E7%9A%84%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.html)



****



**解决单击按钮事件中的冒泡现象**

```javascript
            angular.module('a10_7', [])
            .controller('c10_7', function ($scope) {
                var obj = this;
                obj.click = function (name, $event) {
                    console.log(name + "被触发");
                    if (obj.stopPropagation) {
                        $event.stopPropagation();
                    }
                };
                obj.change = function ($event) {
                    $event.stopPropagation();
                }
                return obj;
            });
```

[解决单击按钮事件中的冒泡现象](https://github.com/aaronzhou0101/Notes/blob/master/slnAngular/Ch10/10-7%20%E8%A7%A3%E5%86%B3%E5%8D%95%E5%87%BB%E6%8C%89%E9%92%AE%E4%BA%8B%E4%BB%B6%E4%B8%AD%E7%9A%84%E5%86%92%E6%B3%A1%E7%8E%B0%E8%B1%A1.html)

---



**释放多余的$watch监测函数**

在$AngularJS$中，当`$watch`函数被直接调用时，将返回一个释放`$watch`绑定的unbind函数。根据这个特性,当需要释放某个多余的`$watch`监测函数时，只需要再次调用这个`$watch`监测函数就可以释放它的监测功能。

```javascript
            angular.module('a10_8', [])
            .controller('c10_8', function ($scope) {
                $scope.num = 0;
                $scope.stopWatch = function () {
                    contentWatch();
                }
                var contentWatch = $scope.$watch('content', function (newVal, oldVal) {
                    if (newVal === oldVal) { return; }
                    $scope.num++; 
                });
            });
```



[释放多余的$watch监测函数](https://github.com/aaronzhou0101/Notes/blob/master/slnAngular/Ch10/10-8%20%E9%87%8A%E6%94%BE%E5%A4%9A%E4%BD%99%E7%9A%84%24watch%E7%9B%91%E6%B5%8B%E5%87%BD%E6%95%B0.html)

---



**解决ng-if中ng-model值无效的问题**

在ng-if方式中，每个包含的元素都拥有自己的作用域，相对于控制器作用域来说，这个作用域属于它的子作用域 ；如果这个元素想绑定控制器中的变量值，必须添加`$parent`标识。

```html
<body>
    <div ng-controller="c10_9" class="frame">
        <div>
            a 的值: {{a}}<br />
            b 的值: {{b}}
        </div>
        <div>
            普通方式: <input type="checkbox" ng-model="a" />
        </div>
        <div ng-if="!a">
            ngIf方式: <input type="checkbox" ng-model="$parent.b" />
        </div>
    </div>
        <script type="text/javascript">
            angular.module('a10_9', [])
            .controller('c10_9', function ($scope) {
                $scope.a = false;
                $scope.b = false;
            });
        </script>
</body>
```



[解决ng-if中ng-model值无效的问题](https://github.com/aaronzhou0101/Notes/blob/master/slnAngular/Ch10/10-9%20%E8%A7%A3%E5%86%B3ng-if%E4%B8%ADng-model%E5%80%BC%E6%97%A0%E6%95%88%E7%9A%84%E9%97%AE%E9%A2%98.html)

