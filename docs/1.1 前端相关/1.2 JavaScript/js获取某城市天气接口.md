[TOC]

# js获取日期
```javascript
    $.ajax({
        type: 'GET',
        url: 'https://free-api.heweather.net/s6/weather/now?location=yantai&key=959b1f49cf104474964d5a9d0328d440',
        dataType: 'JSON',
        error: function () {
            alert('获取天气失败');
        },
        success: function (res) {
            var weather = res.HeWeather6[0].now;
            var imgSrc = "/static/img/weather/" + weather.cond_code + ".png";
            var imgHtml = "<img src='" + imgSrc + "' class='weather-icon'>";
            $('#weather').append(imgHtml + weather.cond_txt + ' ' + weather.tmp + '℃ ' + weather.wind_dir + weather.wind_sc + '级');
        }
    });
```

eg：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/9b3b67f6afcc6a26d449986f25bfecfb?showdoc=.jpg)