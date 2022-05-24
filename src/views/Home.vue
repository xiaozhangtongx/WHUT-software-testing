<template>
  <div>
    <!--limit:最大上传数，on-exceed：超过最大上传数量时的操作  -->
    <el-upload
      class="upload-demo"
      action=""
      :on-change="handleChange"
      :on-remove="handleRemove"
      :on-exceed="handleExceed"
      :limit="limitUpload"
      accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel"
      :auto-upload="false"
    >
      <el-button size="small" type="primary">点击上传</el-button>
      <!-- <div slot="tip" class="el-upload__tip">只 能 上 传 xlsx / xls 文 件</div> -->
    </el-upload>

    <!-- 数据展示 -->
    <el-main>
      <el-table :data="da">
        <el-table-column type="index" width="50"> </el-table-column>
        <el-table-column prop="year" label="测试年份"> </el-table-column>
        <el-table-column prop="month" label="测试月份"> </el-table-column>
        <el-table-column prop="day" label="测试日期"> </el-table-column>
        <el-table-column prop="result" label="预期结果"> </el-table-column>
      </el-table>
    </el-main>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        res: { status: 0, message: '' }, // 用来保存处理结果
        limitUpload: 1,
        fileTemp: null,
        file: null,
        da: [],
        dalen: 0,
        months: [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31], //每月的数据
      }
    },
    methods: {
      handleChange(file, fileList) {
        this.fileTemp = file.raw
        if (this.fileTemp) {
          if (this.fileTemp.type == 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' || this.fileTemp.type == 'application/vnd.ms-excel') {
            this.importfxx(this.fileTemp)
          } else {
            this.$message({
              type: 'warning',
              message: '附件格式错误，请删除后重新上传！',
            })
          }
        } else {
          this.$message({
            type: 'warning',
            message: '请上传附件！',
          })
        }
      },
      handleExceed() {
        this.$message({
          type: 'warning',
          message: '超出最大上传文件数量的限制！',
        })
        return
      },
      handleRemove(file, fileList) {
        this.fileTemp = null
      },
      // 读取excel文档
      importfxx(obj) {
        let _this = this
        let inputDOM = this.$refs.inputer
        // 通过DOM取文件数据
        this.file = event.currentTarget.files[0]
        var rABS = false //是否将文件读取为二进制字符串
        var f = this.file

        var reader = new FileReader()
        //if (!FileReader.prototype.readAsBinaryString) {
        FileReader.prototype.readAsBinaryString = function (f) {
          var binary = ''
          var rABS = false //是否将文件读取为二进制字符串
          var pt = this
          var wb //读取完成的数据
          var outdata
          var reader = new FileReader()
          reader.onload = function (e) {
            var bytes = new Uint8Array(reader.result)
            var length = bytes.byteLength
            for (var i = 0; i < length; i++) {
              binary += String.fromCharCode(bytes[i])
            }
            var XLSX = require('xlsx')
            if (rABS) {
              wb = XLSX.read(btoa(fixdata(binary)), {
                //手动转化
                type: 'base64',
              })
            } else {
              wb = XLSX.read(binary, {
                type: 'binary',
              })
            }
            outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]) //outdata就是你想要的东西
            console.log('未处理的原始数据如下：')
            console.log(outdata)
            //此处可对数据进行处理
            let arr = []
            outdata.map((item) => {
              // 把字符串改成数字
              for (let i in item) {
                item[i] = parseInt(item[i])
              }
              let obj = {}
              obj = Object.assign({}, item)
              // 处理数据
              _this.isLegal(item)
              if (_this.res.status === 1) {
                _this.afterDate(item)
              }
              obj.result = _this.res.message
              arr.push(obj)
            })
            _this.da = arr
            _this.dalen = arr.length
            return arr
          }
          reader.readAsArrayBuffer(f)
        }
        if (rABS) {
          reader.readAsArrayBuffer(f)
        } else {
          reader.readAsBinaryString(f)
        }
      },
      // 判断是否是闰年
      isLeapYear(year) {
        if ((year % 4 === 0 && year % 100 !== 0) || year % 400 === 0) {
          this.months[1] = 29
        } else {
          this.months[1] = 28
        }
      },
      // 判断格式输入是否准确
      isLegal(input) {
        let { year, month, day } = input
        this.isLeapYear(year)
        if (year > 2050 || year < 1900) {
          this.res.status = 0
          this.res.message = '年份输入错误'
        } else if (month > 12 || month < 1) {
          this.res.status = 0
          this.res.message = '月份输入错误'
        } else if (day < 1 || day > this.months[month - 1]) {
          this.res.status = 0
          this.res.message = '日期输入错误'
        } else {
          this.res.status = 1
        }
      },
      //隔一日
      afterDate(date) {
        console.log('arr', date)
        let { year, month, day } = date
        day += 2
        if (day / this.months[month - 1] > 1) {
          day %= this.months[month - 1]
          month++
          if (month > 12) {
            month %= 12
            year++
          }
        }
        this.res.message = year + '年' + month + '月' + day + '日'
      },
    },
  }
</script>
