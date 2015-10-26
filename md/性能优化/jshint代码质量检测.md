# jshint代码质量检测

> JSHint 是一个用于检测JavaScript代码中的错误和潜在问题的工具，以规范编码风格、确保代码的质量。

## 选型

- 基于Gulp构建工具管理前端代码
- 选用gulp-jshint分析前端js代码

## 部分配置代码

```
// npm i gulp-jshint --save-dev
var jshint = require('gulp-jshint');

gulp.task('jshint', function() {
  return gulp.src(['src/**/*.js'])
    .pipe(jshint())
    .pipe(myReporter);
});

var myReporter = map(function (file, cb) {
  if (!file.jshint.success) {
    console.log('JSHINT fail in '+file.path);
    var data = '';
    file.jshint.results.forEach(function (err) {
      if (err) {
        data += '- 行数: ' + err.error.line + ',问题代码: ' + err.error.evidence + ',原因: ' + err.error.reason + '\n';
      }
      
    });
    var targetFileName = file.path.match(/(\w+)\.js$/);
    var targetDir = 'd:\\website\\uui\\docs\\jshint-reporter\\' + targetFileName[1] + '.md';

    writeFile(targetDir, data);
    data = '';

  }
  cb(null, file);
});
```

## 分析结果示例

- 行数: 24,问题代码:      this.plugs || (this.plugs = {}),原因: Expected an assignment or function call and instead saw an expression.
- 行数: 24,问题代码:      this.plugs || (this.plugs = {}),原因: Missing semicolon.
- 行数: 26,问题代码:        throw new Error('plug has exist:'+ name),原因: Missing semicolon.
- 行数: 28,问题代码:      this.plugs[name] = plug,原因: Missing semicolon.
- 行数: 31,问题代码:      var type = options['type'],原因: ['type'] is better written in dot notation.
- 行数: 31,问题代码:      var type = options['type'],原因: Missing semicolon.
- 行数: 32,问题代码:      var plug = this.plugs[type],原因: Missing semicolon.
- 行数: 33,问题代码:      if (!plug) return null,原因: Missing semicolon.
- 行数: 34,问题代码:      return new plug(element, options, viewModel, app),原因: A constructor name should start with an uppercase letter.
- 行数: 73,问题代码:            if(val !='' && val != null){,原因: Use '!==' to compare with ''.


