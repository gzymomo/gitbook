[TOC]

[掘金：叫我詹躲躲：269个JavaScript工具函数，助你提升工作效率（新）](https://juejin.im/post/5edb6c6be51d4578a2555a9b#heading-128)

# 获得项目名称
```javascript
function getPath() {
    var pathName = window.document.location.pathname;
    var projectName = pathName
        .substring(0, pathName.substr(1).indexOf('/') + 1);
    return projectName;
}
```

# 给在0-9的日期加上0
```javascript
function formatNumber(n) {//给在0-9的日期加上0
  n = n.toString()
  return n[1] ? n : '0' + n
}
```

# 动态给Date对象添加新的方法，得到时间yyyy-mm-dd hh:mm:ss
```javascript
Date.prototype.formatDateTime = function () {
    var y = this.getFullYear();
    var m = this.getMonth() + 1;
    var d = this.getDate();
    var hh = this.getHours();
    var mm = this.getMinutes();
    var ss = this.getSeconds();
    return y + "-" + formatNumber(m) + "-" + formatNumber(d) + " "
        + formatNumber(hh) + ":" + formatNumber(mm) + ":"
        + formatNumber(ss);
}
```

# 动态给Date对象添加新的方法,得到yyyy-mm-dd
```javascript
Date.prototype.formatDate = function () {
    var y = this.getFullYear();
    var m = this.getMonth() + 1;
    var d = this.getDate();
    return y + "-" + formatNumber(m) + "-" + formatNumber(d);
}
```

# 字符串工具类
```javascript
var StringUtil = function () {
    /**
     * 判断字符串是否为空
     * @param string 传入字符串
     * @returns {boolean} 返回boolean类型，true表示为空值
     */
    this.isEmptyForString = function (string) {
        if (string === undefined || string === null || string === "") {
            return true;
        }
        return false;
    };
    /**
     * 判断对象是否为空
     * @param object 传入对象
     * @returns {boolean} 如果对象为空则返回true
     */
    this.isEmptyForObject = function (object) {
        if (object === undefined || null === object) {
            return true;
        }
        return false;
    };
}
```
# 通过出生日期得到年龄
```javascript
/**
 * 通过出生日期得到年龄
 * @param str 日期字符串，格式yyyy-MM-dd
 * @returns {number} 年龄数字
 */
function getAge2Birth(str) {
    var strBirthday = str.split(" ");
    var returnAge;
    var strBirthdayArr = strBirthday[0].split("-");
    var birthYear = strBirthdayArr[0];
    var birthMonth = strBirthdayArr[1];
    var birthDay = strBirthdayArr[2];

    d = new Date();
    var nowYear = d.getFullYear();
    var nowMonth = d.getMonth() + 1;
    var nowDay = d.getDate();

    if (nowYear == birthYear) {
        returnAge = 0;// 同年 则为0岁
    } else {
        var ageDiff = nowYear - birthYear; // 年之差
        if (ageDiff > 0) {
            if (nowMonth == birthMonth) {
                var dayDiff = nowDay - birthDay;// 日之差
                if (dayDiff < 0) {
                    returnAge = ageDiff - 1;
                } else {
                    returnAge = ageDiff;
                }
            } else {
                var monthDiff = nowMonth - birthMonth;// 月之差
                if (monthDiff < 0) {
                    returnAge = ageDiff - 1;
                } else {
                    returnAge = ageDiff;
                }
            }
        } else {
            returnAge = -1;// 返回-1 表示出生日期输入错误 晚于今天
        }
    }
    return returnAge;// 返回周岁年龄
}
```

# 将 yyyy-MM-dd hh:mm:ss 格式 或yyyy-MM-dd格式的字符串日期转换成Date类型
```javascript
/**
 * 将 yyyy-MM-dd hh:mm:ss 格式 或yyyy-MM-dd格式的字符串日期转换成Date类型
 * @param dateStr 日期字符串 yyyy-MM-dd hh:mm:ss 格式 或yyyy-MM-dd格式
 * @returns {Date} Date类型
 */
function getDate(dateStr) {
    var str = dateStr.split(" ");
    var strArr = str[0].split("-");
    var Year = strArr[0];
    var Month = strArr[1];
    var Day = strArr[2];
    return new Date(Year, Month, Day);
}
```

# 得到今天的日期
```javascript
/**
 * 得到今天的日期
 * @returns {string} 今日日期字符串
 */
function getNowDate() {
    d = new Date();
    var nowYear = d.getFullYear();
    var nowMonth = d.getMonth() + 1;
    var nowDay = d.getDate();
    return nowYear + "-" + formatNumber(nowMonth) + "-" + formatNumber(nowDay);
}
```

# 得到昨天的日期
```javascript
/**
 * 得到昨天的日期
 * @returns {string} 昨天天气字符串
 */
function getYesterday() {
    var day1 = new Date();
    day1.setDate(day1.getDate() - 1);
    var nowYear = day1.getFullYear();
    var nowMonth = day1.getMonth() + 1;
    var nowDay = day1.getDate();
    return nowYear + "-" + formatNumber(nowMonth) + "-" + formatNumber(nowDay);
}
```

# 获取指定范围区间随机数（min<=取值<=max）
```javascript
/**
 * 获取指定范围区间随机数（min<=取值<=max）
 * @param min 最小值
 * @param max 最大值
 * @returns {*} 随机数
 */
function getRandomNum(min, max) {
    var Range = max - min;
    var Rand = Math.random();
    return (min + Math.round(Rand * Range));
}
```

# 数组工具类
```javascript
/**
 * 数组工具类
 */
var ArrayUtil = function () {
    /**
     * 判断对象是否是一个数组
     * @param value 对象
     * @returns {arg is Array<any>|boolean} boolean类型
     */
    this.isArray = function (value) {
        if (typeof Array.isArray === "function") {
            return Array.isArray(value);
        } else {
            return Object.prototype.toString.call(value) === "[object Array]";
        }
    }
    /**
     * 数组去重
     * @param arr
     * @returns {any[]}
     */
    this.removeArraySameElement = function (arr) {
        var x = new Set(arr);
        return [...x];
    };
    /**
     * 移除数组中空元素
     * @param array
     * @returns {Array}
     */
    this.removeEmptyElement = function (array) {
        let stringUtil = new StringUtil();
        var ar = [];
        if (!stringUtil.isEmptyForObject(array) && array.length > 0) {
            for (var i = 0; i < array.length; i++) {
                if (null != array[i] && array[i] != "" && array[i] != " ") {
                    ar.push(array[i]);
                }
            }
        }
        return ar;
    };
    /**
     * 移除数组元素左右的空白
     * @param array
     * @returns {Array}
     */
    this.removeElementLeftAndRightSpace = function (array) {
        let stringUtil = new StringUtil();
        var ar = [];
        if (!stringUtil.isEmptyForObject(array) && array.length > 0) {
            for (var i = 0; i < array.length; i++) {
                ar.push(array[i].trim());
            }
        }
        return ar;
    };
    /**
     * 移除数组指定元素
     * @param array 传入数组
     * @param element 需要移除的数组元素
     * @returns {Array}
     */
    this.removeArrayElement = function (array, element) {
        let stringUtil = new StringUtil();
        var arrayCopy = [];
        if (!stringUtil.isEmptyForObject(array) && array.length > 0) {
            for (var i = 0; i < array.length; i++) {
                if (array[i] != element) {
                    arrayCopy.push(array[i]);
                }
            }
        }
        return arrayCopy;
    };
    /**
     * 判断数组中元素是否相同
     * @param array1 数组1
     * @param array2 数组2
     * @returns {boolean} boolean类型
     */
    this.arraySame = function (array1, array2) {
        let stringUtil = new StringUtil();
        if (!stringUtil.isEmptyForObject(array1) && array1.length > 0 && !stringUtil.isEmptyForObject(array2) && array2.length > 0) {
            if (array1.length != array2.length) {
                return false;
            }
            var sign = 0;
            for (var i = 0; i < array1.length; i++) {
                for (var j = 0; j < array2.length; j++) {
                    if (array1[i] === array2[j]) {
                        sign++;
                        break;
                    }
                }
            }
            if (sign === array1.length) {
                return true;
            }
        }
        return false;
    };
    /**
     * 从数组中1中删除数组2包含的元素
     * @param array1 数组1
     * @param array2 数组2
     * @returns {Array} 新数组
     */
    this.removeSameArray = function (array1, array2) {
        let stringUtil = new StringUtil();
        var array = array1;
        if (!stringUtil.isEmptyForObject(array2) && array2.length > 0) {
            for (let i = 0; i < array2.length; i++) {
                array = this.removeArrayElement(array, array2[i]);
            }
        }
        return array;
    };
    /**
     * 判断数组中是否包含某个元素
     * @param array 数组
     * @param element 元素
     * @returns {boolean} boolean类型
     */
    this.arrayIncludeElement = function (array, element) {
        let stringUtil = new StringUtil();
        if (stringUtil.isEmptyForObject(array)) {
            return false;
        }
        var bool = false;
        for (var i = 0; i < array.length; i++) {
            if (array[i] === element) {
                bool = true;
                break;
            }
        }
        return bool;
    };

    /**
     * 将数组转化成为指定字符隔开的字符串
     * @param array 数组
     * @param charact 分隔符
     * @returns {string} 拼接后的字符串
     */
    this.array2CharSepStr = function (array, charact) {
        let stringUtil = new StringUtil();
        var arrayStr = "";
        if (!stringUtil.isEmptyForObject(array) && array.length > 0) {
            for (var i = 0; i < array.length; i++) {
                if (i === 0) {
                    arrayStr = arrayStr + array[i];
                } else {
                    arrayStr = arrayStr + charact + array[i];
                }
            }
        }
        return arrayStr;
    };
    /**
     * 将字符串转换成数组
     * @param str 指定分割符的字符串
     * @param charact 分隔符
     * @returns {string|*|null|string[]|never}
     */
    this.charSepStr2Array = function (str, charact) {
        let stringUtil = new StringUtil();
        if (!stringUtil.isEmptyForString(str)) {
            return str.split(charact);
        }
        return "";
    };
    /**
     * 将其他数组中的元素添加进当前数组中
     * @param arrayIn 被添加的数组（当前数组）
     * @param arrayOther 要添加的数组（其他数组）
     * @returns {any[]|Array} 数组
     */
    this.addOtherAarryInArray = function (arrayIn, arrayOther) {
        var arrayInCopy = [];
        if (!this.arrayIsEmpty(arrayIn)) {
            arrayInCopy = arrayIn;
        }
        if (!this.arrayIsEmpty(arrayOther)) {
            for (let i = 0; i < arrayOther.length; i++) {
                arrayInCopy.push(arrayOther[i]);
            }
            return this.removeArraySameElement(arrayInCopy);
        }
        return arrayInCopy;
    };
    /**
     * 判断数组是否为空
     * @param array 数组
     * @returns {boolean} boolean类型
     */
    this.arrayIsEmpty = function (array) {
        if (array === undefined || array === null || array.length === 0) {
            return true;
        }
        return false;
    }
};
```

# 日期工具类
```javascript
/**
 * 针对Ext的工具类(日期工具类)
 */
var DateUtil = function () {
    /***
     * 获得当前时间
     */
    this.getCurrentDate = function () {
        return new Date();
    };
    /***
     * 获得本周起止时间
     */
    this.getCurrentWeek = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //返回date是一周中的某一天
        var week = currentDate.getDay();
        //返回date是一个月中的某一天
        var month = currentDate.getDate();

        //一天的毫秒数
        var millisecond = 1000 * 60 * 60 * 24;
        //减去的天数
        var minusDay = week != 0 ? week - 1 : 6;
        //alert(minusDay);
        //本周 周一
        var monday = new Date(currentDate.getTime() - (minusDay * millisecond));
        //本周 周日
        var sunday = new Date(monday.getTime() + (6 * millisecond));
        //添加本周时间
        startStop.push(monday);//本周起始时间
        //添加本周最后一天时间
        startStop.push(sunday);//本周终止时间
        //返回
        return startStop;
    };

    /***
     * 获得本月的起止时间
     */
    this.getCurrentMonth = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //获得当前月份0-11
        var currentMonth = currentDate.getMonth();
        //获得当前年份4位年
        var currentYear = currentDate.getFullYear();
        //求出本月第一天
        var firstDay = new Date(currentYear, currentMonth, 1);


        //当为12月的时候年份需要加1
        //月份需要更新为0 也就是下一年的第一个月
        if (currentMonth == 11) {
            currentYear++;
            currentMonth = 0;//就为
        } else {
            //否则只是月份增加,以便求的下一月的第一天
            currentMonth++;
        }


        //一天的毫秒数
        var millisecond = 1000 * 60 * 60 * 24;
        //下月的第一天
        var nextMonthDayOne = new Date(currentYear, currentMonth, 1);
        //求出上月的最后一天
        var lastDay = new Date(nextMonthDayOne.getTime() - millisecond);

        //添加至数组中返回
        startStop.push(firstDay);
        startStop.push(lastDay);
        //返回
        return startStop;
    };

    /**
     * 得到本季度开始的月份
     * @param month 需要计算的月份
     ***/
    this.getQuarterSeasonStartMonth = function (month) {
        var quarterMonthStart = 0;
        var spring = 0; //春
        var summer = 3; //夏
        var fall = 6;   //秋
        var winter = 9;//冬
        //月份从0-11
        if (month < 3) {
            return spring;
        }

        if (month < 6) {
            return summer;
        }

        if (month < 9) {
            return fall;
        }

        return winter;
    };

    /**
     * 获得该月的天数
     * @param year年份
     * @param month月份
     * */
    this.getMonthDays = function (year, month) {
        //本月第一天 1-31
        var relativeDate = new Date(year, month, 1);
        //获得当前月份0-11
        var relativeMonth = relativeDate.getMonth();
        //获得当前年份4位年
        var relativeYear = relativeDate.getFullYear();

        //当为12月的时候年份需要加1
        //月份需要更新为0 也就是下一年的第一个月
        if (relativeMonth == 11) {
            relativeYear++;
            relativeMonth = 0;
        } else {
            //否则只是月份增加,以便求的下一月的第一天
            relativeMonth++;
        }
        //一天的毫秒数
        var millisecond = 1000 * 60 * 60 * 24;
        //下月的第一天
        var nextMonthDayOne = new Date(relativeYear, relativeMonth, 1);
        //返回得到上月的最后一天,也就是本月总天数
        return new Date(nextMonthDayOne.getTime() - millisecond).getDate();
    };

    /**
     * 获得本季度的起止日期
     */
    this.getCurrentSeason = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //获得当前月份0-11
        var currentMonth = currentDate.getMonth();
        //获得当前年份4位年
        var currentYear = currentDate.getFullYear();
        //获得本季度开始月份
        var quarterSeasonStartMonth = this.getQuarterSeasonStartMonth(currentMonth);
        //获得本季度结束月份
        var quarterSeasonEndMonth = quarterSeasonStartMonth + 2;

        //获得本季度开始的日期
        var quarterSeasonStartDate = new Date(currentYear, quarterSeasonStartMonth, 1);
        //获得本季度结束的日期
        var quarterSeasonEndDate = new Date(currentYear, quarterSeasonEndMonth, this.getMonthDays(currentYear, quarterSeasonEndMonth));
        //加入数组返回
        startStop.push(quarterSeasonStartDate);
        startStop.push(quarterSeasonEndDate);
        //返回
        return startStop;
    };

    /***
     * 得到本年的起止日期
     *
     */
    this.getCurrentYear = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //获得当前年份4位年
        var currentYear = currentDate.getFullYear();

        //本年第一天
        var currentYearFirstDate = new Date(currentYear, 0, 1);
        //本年最后一天
        var currentYearLastDate = new Date(currentYear, 11, 31);
        //添加至数组
        startStop.push(currentYearFirstDate);
        startStop.push(currentYearLastDate);
        //返回
        return startStop;
    };

    /**
     * 返回上一个月的第一天Date类型
     * @param year 年
     * @param month 月
     **/
    this.getPriorMonthFirstDay = function (year, month) {
        //年份为0代表,是本年的第一月,所以不能减
        if (month == 0) {
            month = 11;//月份为上年的最后月份
            year--;//年份减1
            return new Date(year, month, 1);
        }
        //否则,只减去月份
        month--;
        return new Date(year, month, 1);
        ;
    };

    /**
     * 获得上一月的起止日期
     * ***/
    this.getPreviousMonth = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //获得当前月份0-11
        var currentMonth = currentDate.getMonth();
        //获得当前年份4位年
        var currentYear = currentDate.getFullYear();
        //获得上一个月的第一天
        var priorMonthFirstDay = this.getPriorMonthFirstDay(currentYear, currentMonth);
        //获得上一月的最后一天
        var priorMonthLastDay = new Date(priorMonthFirstDay.getFullYear(), priorMonthFirstDay.getMonth(), this.getMonthDays(priorMonthFirstDay.getFullYear(), priorMonthFirstDay.getMonth()));
        //添加至数组
        startStop.push(priorMonthFirstDay);
        startStop.push(priorMonthLastDay);
        //返回
        return startStop;
    };

    //获得上月的月份
    this.getLastMoth = function () {
        var lastMonthStartAndEnd = this.getPreviousMonth();
        var lastMonthStart = lastMonthStartAndEnd[0];
        var nowYear = lastMonthStart.getFullYear();
        var lastMonth = lastMonthStart.getMonth();
        var lastDay = lastMonthStart.getDay();
        console.log(lastMonth);
        return nowYear + "-" + (lastMonth + 1) + "-" + lastDay;
    };

    //获得本月的月份
    this.getNowMoth = function () {
        var nowMonthStartAndEnd = this.getCurrentMonth();
        var nowMonthStart = nowMonthStartAndEnd[0];
        var nowYear = nowMonthStart.getFullYear();
        var nowMonth = nowMonthStart.getMonth();
        var nowDay = nowMonthStart.getDay();
        return nowYear + "-" + (nowMonth + 1) + "-" + nowDay;
    };

    /**
     * 获得上一周的起止日期
     * **/
    this.getPreviousWeek = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //返回date是一周中的某一天
        var week = currentDate.getDay();
        //返回date是一个月中的某一天
        var month = currentDate.getDate();
        //一天的毫秒数
        var millisecond = 1000 * 60 * 60 * 24;
        //减去的天数
        var minusDay = week != 0 ? week - 1 : 6;
        //获得当前周的第一天
        var currentWeekDayOne = new Date(currentDate.getTime() - (millisecond * minusDay));
        //上周最后一天即本周开始的前一天
        var priorWeekLastDay = new Date(currentWeekDayOne.getTime() - millisecond);
        //上周的第一天
        var priorWeekFirstDay = new Date(priorWeekLastDay.getTime() - (millisecond * 6));

        //添加至数组
        startStop.push(priorWeekFirstDay);
        startStop.push(priorWeekLastDay);

        return startStop;
    };

    /**
     * 得到上季度的起始日期
     * year 这个年应该是运算后得到的当前本季度的年份
     * month 这个应该是运算后得到的当前季度的开始月份
     * */
    this.getPriorSeasonFirstDay = function (year, month) {
        var quarterMonthStart = 0;
        var spring = 0; //春
        var summer = 3; //夏
        var fall = 6;   //秋
        var winter = 9;//冬
        //月份从0-11
        switch (month) {//季度的其实月份
            case spring:
                //如果是第一季度则应该到去年的冬季
                year--;
                month = winter;
                break;
            case summer:
                month = spring;
                break;
            case fall:
                month = summer;
                break;
            case winter:
                month = fall;
                break;

        }
        ;

        return new Date(year, month, 1);
    };

    /**
     * 得到上季度的起止日期
     * **/
    this.getPreviousSeason = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //获得当前月份0-11
        var currentMonth = currentDate.getMonth();
        //获得当前年份4位年
        var currentYear = currentDate.getFullYear();
        //上季度的第一天
        var priorSeasonFirstDay = this.getPriorSeasonFirstDay(currentYear, currentMonth);
        //上季度的最后一天
        var priorSeasonLastDay = new Date(priorSeasonFirstDay.getFullYear(), priorSeasonFirstDay.getMonth() + 2, this.getMonthDays(priorSeasonFirstDay.getFullYear(), priorSeasonFirstDay.getMonth() + 2));
        //添加至数组
        startStop.push(priorSeasonFirstDay);
        startStop.push(priorSeasonLastDay);
        return startStop;
    };

    /**
     * 得到去年的起止日期
     * **/
    this.getPreviousYear = function () {
        //起止日期数组
        var startStop = new Array();
        //获取当前时间
        var currentDate = this.getCurrentDate();
        //获得当前年份4位年
        var currentYear = currentDate.getFullYear();
        currentYear--;
        var priorYearFirstDay = new Date(currentYear, 0, 1);
        var priorYearLastDay = new Date(currentYear, 11, 1);
        //添加至数组
        startStop.push(priorYearFirstDay);
        startStop.push(priorYearLastDay);
        return startStop;
    };
};
```

# 隐藏银行卡中间数字方法
```javascript
/**
 * 隐藏银行卡中间数字方法
 * @param cardNumber
 * @returns {*|string}
 */
function cardSubstring(cardNumber) {
    let stringUtil = new StringUtil();
    var text = cardNumber;
    if (!stringUtil.isEmptyForString(cardNumber) && cardNumber.length >= 8) {
        text = cardNumber.substring(0, 4);
        text = text + "***********";
        text = text + cardNumber.substring(cardNumber.length - 4, cardNumber.length);
    }
    return text;
}
```

# 判断字符串以xxx开头，以xxx结尾
```javascript
var startWith = function (value, start) {
    var reg = new RegExp("^" + start);
    return reg.test(value)
}
// 判断字符串是否是以end结尾
var endWith = function (value, end) {
    var reg = new RegExp(end + "$");
    return reg.test(value)
}
```

# 1. 获取文件后缀名

使用场景：上传文件判断后缀名

```js
/**
 * 获取文件后缀名
 * @param {String} filename
 */
 export function getExt(filename) {
    if (typeof filename == 'string') {
        return filename
            .split('.')
            .pop()
            .toLowerCase()
    } else {
        throw new Error('filename must be a string type')
    }
}
```

使用方式

```js
getExt("1.mp4") //->mp4
```

# 2. 复制内容到剪贴板

```js
export function copyToBoard(value) {
    const element = document.createElement('textarea')
    document.body.appendChild(element)
    element.value = value
    element.select()
    if (document.execCommand('copy')) {
        document.execCommand('copy')
        document.body.removeChild(element)
        return true
    }
    document.body.removeChild(element)
    return false
}
```

使用方式:

```js
//如果复制成功返回true
copyToBoard('lalallala')
```

原理：

1. 创建一个textare元素并调用select()方法选中
2. document.execCommand('copy')方法，拷贝当前选中内容到剪贴板。

# 3. 休眠多少毫秒

```js
/**
 * 休眠xxxms
 * @param {Number} milliseconds
 */
export function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
}

//使用方式
const fetchData=async()=>{
	await sleep(1000)
}
```

# 4. 生成随机字符串

```js
/**
 * 生成随机id
 * @param {*} length
 * @param {*} chars
 */
export function uuid(length, chars) {
    chars =
        chars ||
        '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
    length = length || 8
    var result = ''
    for (var i = length; i > 0; --i)
        result += chars[Math.floor(Math.random() * chars.length)]
    return result
}
```

使用方式

```js
//第一个参数指定位数，第二个字符串指定字符，都是可选参数，如果都不传，默认生成8位
uuid()  
```

使用场景：用于前端生成随机的ID,毕竟现在的Vue和React都需要绑定key

# 5. 简单的深拷贝

```js
/**
 *深拷贝
 * @export
 * @param {*} obj
 * @returns
 */
export function deepCopy(obj) {
    if (typeof obj != 'object') {
        return obj
    }
    if (obj == null) {
        return obj
    }
    return JSON.parse(JSON.stringify(obj))
}
```

缺陷：只拷贝对象、数组以及对象数组，对于大部分场景已经足够

```js
const person={name:'xiaoming',child:{name:'Jack'}}
deepCopy(person) //new person
复制代码
```

# 6. 数组去重

```js
/**
 * 数组去重
 * @param {*} arr
 */
export function uniqueArray(arr) {
    if (!Array.isArray(arr)) {
        throw new Error('The first parameter must be an array')
    }
    if (arr.length == 1) {
        return arr
    }
    return [...new Set(arr)]
}
```

原理是利用Set中不能出现重复元素的特性

```js
uniqueArray([1,1,1,1,1])//[1]
```

# 7. 对象转化为FormData对象

```js
/**
 * 对象转化为formdata
 * @param {Object} object
 */

 export function getFormData(object) {
    const formData = new FormData()
    Object.keys(object).forEach(key => {
        const value = object[key]
        if (Array.isArray(value)) {
            value.forEach((subValue, i) =>
                formData.append(key + `[${i}]`, subValue)
            )
        } else {
            formData.append(key, object[key])
        }
    })
    return formData
}
```

使用场景：上传文件时我们要新建一个FormData对象，然后有多少个参数就append多少次，使用该函数可以简化逻辑

使用方式：

```js
let req={
    file:xxx,
    userId:1,
    phone:'15198763636',
    //...
}
fetch(getFormData(req))
```

# 8.保留到小数点以后n位

```js
// 保留小数点以后几位，默认2位
export function cutNumber(number, no = 2) {
    if (typeof number != 'number') {
        number = Number(number)
    }
    return Number(number.toFixed(no))
}
```

使用场景：JS的浮点数超长，有时候页面显示时需要保留2位小数


