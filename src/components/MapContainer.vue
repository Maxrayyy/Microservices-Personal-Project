<script setup>
import { ref, onMounted, onUnmounted, watch} from "vue"
import axios from "axios"
import { ElMessage } from 'element-plus'

const cityName = ref("")
//https://www.tianapi.com  天行数据秘钥
const API_KEY = "fae142356d8b697bd64380a07d1e5ab0"
//https://www.mxnzp.com  ROLL数据平台秘钥
const APP_ID = "stmpxpklrxjdinfv"
const APP_SECRET = "S6xXL5tEZLblDxi9glrznI5OZIZOtLUa"
//https://eolink.o.apispace.com  请求头部X-APISpace-Token
const X_APISPACE_TOKEN = "ckh2f2g5uzgbz859jl6yhfxfzyuqm9d7"
//百度地图秘钥  
const BAIDU_MAP_API_KEY = "Le0uAtMIlPwmIylegndgXVhpYFDAHUuE"

const tourist_attraction_num = 10  //地区景点条数为 10
const cultural_tourism_news_num = 10  //新闻条数为 10
const weather_forecast_type = 7  // 1表示当天天气，7表示7天天气

const wikipediaInfo = ref([])
//该城市旅游景点信息
const tourist_attraction = ref([])
//该城市文旅新闻信息
const cultural_tourism_news = ref([])
//该城市天气预报信息
const weather_forecast = ref([])
//该城市油价信息
const oil_price = ref([])
//该城市空气质量信息
const air_condition = ref([])
//该城市行政区划代码
const areacode = ref("")
//城市所属省份及其区信息
const code = ref('')
const province = ref('')
const area = ref([])

const showCityInfo = ref(false)

const changeProviceKey = (old_province) => {
  //标格式：(提取前两个字)
  //安徽、北京、重庆、福建、甘肃、广东、广西、贵州、海南、
  //河北、黑龙江、河南、湖北、湖南、江苏、江西、吉林、辽宁、
  //内蒙古、宁夏、青海、陕西、上海、山东、山西、四川、天津、西藏、新疆、云南、浙江
  //需要特殊转化的省份：（提取前三个字）
  //新疆维吾尔自治区 内蒙古自治区 黑龙江省 广西壮族自治区 宁夏回族自治区 西藏自治区
  if(old_province == '新疆维吾尔自治区' || old_province == '内蒙古自治区' || old_province == '黑龙江省' || old_province == '广西壮族自治区' || old_province == '宁夏回族自治区' || old_province == '西藏自治区'){
    return old_province.substr(0,3);
  }
  else{
    return old_province.substr(0,2);
  }
}

//获取城市所属省份及其区信息
const getProvinceArea = async () => {
  try {
    const response = await axios.get(`https://www.mxnzp.com/api/address/search?type=1&value=${cityName.value}&app_id=${APP_ID}&app_secret=${APP_SECRET}`)
        code.value = response.data.data[0].code;
        province.value = changeProviceKey(response.data.data[0].name);
        area.value = response.data.data[0].pchilds[0].cchilds;
        // 请求成功时的处理
        console.log("获取省份及区信息成功：",response.data.data[0])

        setTimeout(() => {
          getOilPrice()
        }, 1000)
  } catch (error) {
    ElMessage({
      showClose: true,
      message: '获取省份及区信息失败',
      type: 'error',
    })
  }
}
//获取城市行政区划代码
const getCityAreacode = async () => {
  try{
    const response = await axios.get(`https://eolink.o.apispace.com/xzqcx/api/v1/administrative_area`,{
      headers: {
        'X-APISpace-Token': X_APISPACE_TOKEN
      },
      params: {
        code_or_name: cityName.value
      }
    })
    areacode.value = response.data.data[0].code 
  } catch (error) {
    ElMessage({
      showClose: true,
      message: '获取行政区划代码失败',
      type: 'error',
    })
  }
}
//获取城市空气质量信息
const getAirConditon = async () => {
  try {
    const response = await axios.get(`https://apis.tianapi.com/aqi/index?key=${API_KEY}&area=${cityName.value}`)
      console.log("空气质量信息：",response.data.result)
      air_condition.value.push(response.data.result)
  } catch (error) {
    ElMessage({
      showClose: true,
      message: '获取空气质量信息失败',
      type: 'error',
    })
  }
}

//获取城市油价信息
const getOilPrice = async () => {
  try {
    const response = await axios.get(`https://www.mxnzp.com/api/oil/search?province=${province.value}&app_id=${APP_ID}&app_secret=${APP_SECRET}`)
    console.log("油价信息：",response.data.data)
    oil_price.value = response.data.data;
  } catch (error) {
    ElMessage({
      showClose: true,
      message: '获取油价信息失败',
      type: 'error',
    })
  }
}
//获取城市天气预报信息
const getWeatherForecast = async () => {
  try{
    const response = await axios.get(`https://apis.tianapi.com/tianqi/index?key=${API_KEY}&city=${cityName.value}&type=${weather_forecast_type}`)
    console.log("天气预报信息：",response.data.result)
    weather_forecast.value = response.data.result.list
  } catch (error) {
    console.error('Error:', error);
    ElMessage({
      showClose: true,
      message: '获取天气预报信息失败',
      type: 'error',
    })
  }
}
//获取城市旅游景点信息
const getTouristAttraction = async () => {
  try{
    const response = await axios.get(`https://apis.tianapi.com/scenic/index?key=${API_KEY}&city=${cityName.value}&num=${tourist_attraction_num}`)
    console.log("景点信息：",response.data)
    tourist_attraction.value = response.data.result.list

  } catch (error) {
    ElMessage({
      showClose: true,
      message: '获取旅游景点信息失败',
      type: 'error',
    })
  }
}
//获取城市文旅新闻信息
const getCulturalTourismNews = async () => {
  try{
    const response = await axios.get(`https://apis.tianapi.com/travel/index?key=${API_KEY}&num=${cultural_tourism_news_num}`)
    console.log("文旅新闻信息：",response.data.result.newslist)
    cultural_tourism_news.value = response.data.result.newslist
  }catch (error) {
    ElMessage({
      showClose: true,
      message: '获取文旅新闻信息失败',
      type: 'error',
    })
  }
}

const fetchWikipediaInfo = async () => {
  if (cityName.value && cityName.value !== '请输入城市名称') {
    const response = await axios.get(`https://zh.wikipedia.org/w/api.php`, {
      params: {
        format: 'json',
        action: 'query',
        generator: 'search',
        gsrnamespace: 0,
        gsrlimit: 10,
        prop: 'pageimages|extracts',
        pilimit: 'max',
        exintro: true,
        explaintext: true,
        exsentences: 1,
        exlimit: 'max',
        origin: '*',
        gsrsearch: cityName.value
      }
    })
    const pages = response.data.query.pages;
    wikipediaInfo.value = Object.keys(pages).map((pageid) => {
      const page = pages[pageid]; 
      const title = page.title;
      const extract = page.extract;
      const imgsrc = page.thumbnail ? page.thumbnail.source : '';
      const listcontent = imgsrc ? `<img src="${imgsrc}">` + extract : extract;
      const href = 'http://zh.wikipedia.org/wiki/' + encodeURIComponent(title);
      return { title, content: listcontent, url: href, pageid };
    });
    console.log("维基百科信息：",wikipediaInfo.value)
    // showResults.value = true;
  }
  else {
    cityName.value = '请输入城市名称';
  }
}
//开始搜索
const searchCity = async () => {
  await getProvinceArea();
  await getCityAreacode();
  await getWeatherForecast();
  await getAirConditon();
  await getTouristAttraction();
  await getCulturalTourismNews();
  await fetchWikipediaInfo();
  showCityInfo.value = true;
};

</script>
<template>
  <div class="app-container">
    <!-- 搜索部分 -->
    <div class="search-section">
      <div class="search-container">
        <input
          type="text"
          v-model="cityName"
          @keyup.enter="searchCity"
          placeholder="请输入城市名称"
          class="city-input"
        />
        <button @click="searchCity" class="search-button">搜索</button>
      </div>
    </div>

    <!-- 第一行：地图和天气 -->
    <div class="row" v-if="showCityInfo">
      <!-- 地图展示 -->
      <div class="column map">
        <h3>地图位置</h3>
        <baidu-map class="bm-view" :zoom="12" :center="cityName" :scroll-wheel-zoom="true" inertial-dragging>
        </baidu-map>
      </div>

      <!-- 天气预报 -->
      <div class="column weather">
        <h3>天气预报</h3>
        <div class="table-container">
          <table class="weather-table">
            <thead>
              <tr>
                <th>日期</th>
                <th>星期</th>
                <th>天气</th>
                <th>实温度</th>
                <th>最低温度</th>
                <th>最高温度</th>
                <th>风向</th>
                <th>风速</th>
                <th>风力等级</th>
                <th>日出</th>
                <th>日落</th>
                <th>湿度</th>
                <th>穿衣出行提示</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in weather_forecast" :key="item.date">
                <td>{{ item.date }}</td>
                <td>{{ item.week }}</td>
                <td>{{ item.weather }}</td>
                <td>{{ item.real }}</td>
                <td>{{ item.lowest }}</td>
                <td>{{ item.highest }}</td>
                <td>{{ item.wind }}</td>
                <td>{{ item.windspeed }}</td>
                <td>{{ item.windsc }}</td>
                <td>{{ item.sunrise }}</td>
                <td>{{ item.sunset }}</td>
                <td>{{ item.humidity }}</td>
                <td>{{ item.tips }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- 第二行：省市信息和旅游景点 -->
    <div class="row" v-if="showCityInfo">
      <!-- 省市信息 -->
      <div class="column city-info">
        <h3>城市区划信息</h3>
        <div class="info-content">
          <p>城市名称: {{ cityName }}</p>
          <p>省份: {{ province }}</p>
          <p>行政区划代码: {{ areacode }}</p>
        </div>
        <div class="area-info">
          <h4>行政区划</h4>
          <div class="scroll-container">
            <table class="area-table">
              <thead>
                <tr>
                  <th>名称</th>
                  <th>代码</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in area" :key="item.code">
                  <td>{{ item.name }}</td>
                  <td>{{ item.code }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- 旅游景点 -->
      <div class="column attractions">
        <h3>旅游景点</h3>
        <div class="scroll-container">
          <table class="attraction-table">
            <thead>
              <tr>
                <th class="attraction-name">名称</th>
                <th class="attraction-content">简介</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(attraction, index) in tourist_attraction" :key="index">
                <td class="attraction-name">{{ attraction.name }}</td>
                <td class="attraction-content">{{ attraction.content }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- 第三行：空气质量和油价信息 -->
    <div class="row" v-if="showCityInfo">
      <!-- 空气质量 -->
      <div class="column air-quality">
        <h3>空气质量</h3>
        <div class="airquality-container">
        <p style="text-align: right; font-size: 12px;">污染物单位：μg/m³</p>
        <table class="airquality-table">
          <thead>
              <tr>
                <th>空气质量指数(AQI)</th>
                <th>空气质量等级</th>
                <th>PM2.5</th>
                <th>PM10</th>
                <th>SO2</th>
                <th>NO2</th>
                <th>CO</th>
                <th>O3</th>
                <th>O3_8h</th>
                <th>首要污染物</th>
              </tr>
          </thead>
          <tbody>
              <tr>
                <td>{{ air_condition[0].aqi }}</td>
                <td>{{ air_condition[0].quality }}</td>
                <td>{{ air_condition[0].pm2_5 }}</td>
                <td>{{ air_condition[0].pm10 }}</td>
                <td>{{ air_condition[0].so2 }}</td>
                <td>{{ air_condition[0].no2 }}</td>
                <td>{{ air_condition[0].co }}</td>
                <td>{{ air_condition[0].o3 }}</td>
                <td>{{ air_condition[0].o3_8h }}</td>
                <td>{{ air_condition[0].primary_pollutant }}</td>
              </tr>
          </tbody>
          </table>
          </div>
      </div>
      <!-- 油价信息 -->
      <div class="column oil-price">
      <h3>油价信息</h3>
          <p style="text-align: left; font-size: 12px;">省份: {{ oil_price.province }}</p>
          <table class="oil-price-table">
          <thead>
            <tr>
              <th>油号</th>
              <th>价格 (元/升)</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>0号柴油</td>
              <td>{{ oil_price.t0 }}</td>
            </tr>
            <tr>
              <td>89号汽油</td>
              <td>{{ oil_price.t89 }}</td>
            </tr>
            <tr>
              <td>92号汽油</td>
              <td>{{ oil_price.t92 }}</td>
            </tr>
            <tr>
              <td>95号汽油</td>
              <td>{{ oil_price.t95 }}</td>
            </tr>
            <tr>
              <td>98号汽油</td>
              <td>{{ oil_price.t98 }}</td>
            </tr>
            </tbody>
          </table>
      </div>
    </div>

    <!-- 文旅新闻 -->
    <div class="row" v-if="showCityInfo">
      <div class="column full-width">
        <h3>文旅新闻</h3>
        <div class="table-container">
          <table class="news-table">
            <thead>
              <tr>
                <th>标题</th>
                <th>来源</th>
                <th>发布时间</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="news in cultural_tourism_news" :key="news.id">
                <td>
                  <a :href="news.url" target="_blank" class="news-title">{{ news.title }}</a>
                </td>
                <td>{{ news.source }}</td>
                <td>{{ news.ctime }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- 维基百科部分 -->
    <div class="row" v-if="showCityInfo">
      <div class="column full-width">
        <h3>维基百科</h3>
        <div class="wikipedia-container">
          <div v-for="item in wikipediaInfo" :key="item.pageid" class="wiki-item">
            <h4 class="wiki-title">{{ item.title }}</h4>
            <div class="wiki-content" v-html="item.content"></div>
            <a :href="item.url" target="_blank" class="wiki-link">查看更多</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  font-family: Arial, sans-serif;
  padding: 20px;
  background-image: url('@/assets/images/background.jpg');
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
  min-height: 100vh; /* 确保容器至少占满整个视口高度 */
}

.search-section {
  margin-bottom: 30px;
  display: flex;
  justify-content: center;
}

.search-container {
  display: flex;
  align-items: center;
  max-width: 600px;
  width: 100%;
}

.city-input {
  flex-grow: 1;
  padding: 12px 20px;
  font-size: 16px;
  border: 2px solid #007bff;
  border-radius: 5px 0 0 5px;
  outline: none;
}

.search-button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #007bff;
  color: white;
  border: 2px solid #007bff;
  border-radius: 0 5px 5px 0;
  cursor: pointer;
  transition: background-color 0.3s;
}

.search-button:hover {
  background-color: #0056b3;
}

.row {
  display: flex;
  margin-bottom: 20px;
}

h3 {
  text-align: center;
}

.column {
  background-color: rgba(240, 240, 240, 0.6);
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.map {
  width: 35%;
  margin-right: 3%;
}

.map-container {
  width: 100%;
  height: 400px;
}

.bm-view {
  width: 100%;
  height: 300px;
}

.weather {
  width: 62%;
}

.city-info, .attractions {
  width: 48.5%;
  height: 440px; /* 设置固定高度 */
  overflow: hidden; /* 防止内容溢出 */
  display: flex;
  flex-direction: column;
}

.city-info {
  margin-right: 3%;
}

.map-container {
  width: 100%;
  height: 300px;
  background-color: #e0e0e0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.full-width {
  width: 100%;
}

/* 修改第三行的样式 */
.air-quality {
  width: 75%; /* 70% 减去间隔的 2% */
  margin-right: 2%;
}

.oil-price {
  width: 23%;
}

.table-container {
  overflow-x: auto;
  max-height: 350px; /* 减小高度以适应更小的字体 */
  margin-top: 20px;
  overflow-y: auto;
}

.weather-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px; /* 减小字体大小 */
  white-space: nowrap; /* 防止文字换行 */
}

.weather-table th,
.weather-table td {
  border: 1px solid #ddd;
  padding: 6px; /* 减小内边距 */
  text-align: center;
}

.weather-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.weather-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.weather-table tr:hover {
  background-color: #f5f5f5;
}

.city-info {
  display: flex;
  flex-direction: column;
}

.info-content {
  margin-bottom: 15px;
}

.area-info {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.scroll-container {
  flex-grow: 1;
  overflow-y: auto;
  border: 1px solid #ddd;
  border-radius: 5px;
  max-height: 250px; /* 限制区域表格的最大高度 */
}

.area-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 11px; /* 减小字体大小 */
}

.area-table th,
.area-table td {
  border: 1px solid #ddd;
  padding: 6px; /* 减小内边距 */
  text-align: left;
}

.area-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.area-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.area-table tr:hover {
  background-color: #f5f5f5;
}

.attractions {
  display: flex;
  flex-direction: column;
}

.attractions .scroll-container {
  flex-grow: 1;
  overflow-y: auto;
  border: 1px solid #ddd;
  border-radius: 5px;
  max-height: 400px; /* 调整高度以匹配城市信息部分 */
}

.attraction-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 11px;
}

.attraction-table th,
.attraction-table td {
  border: 1px solid #ddd;
  padding: 6px;
  text-align: left;
  overflow: hidden;
  text-overflow: ellipsis;
}

.attraction-name {
  text-align: center;
  width: 18%; /* 调整名称列的宽度 */
  white-space: nowrap; /* 防止名称换行 */
}

.attraction-content {
  width: 82%; /* 调整简介列的宽度 */
}

.attraction-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.attraction-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.attraction-table tr:hover {
  background-color: #f5f5f5;
}

/* 调整城市信息和景点列的样式 */
.city-info, .attractions {
  width: 48.5%;
  height: 440px; /* 设置固定高度 */
  overflow: hidden; /* 防止内容溢出 */
  display: flex;
  flex-direction: column;
}

.city-info .scroll-container,
.attractions .scroll-container {
  flex-grow: 1;
  overflow-y: auto;
}

.airquality-table {
  width: 100%;
  margin: 50px auto;
  border-collapse: collapse;
  font-size: 12px; /* 减小字体大小 */
  white-space: nowrap; /* 防止文字换行 */
}

.airquality-table th,
.airquality-table td {
  border: 1px solid #ddd;
  padding: 6px; /* 减小内边距 */
  text-align: center;
}

.airquality-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.airquality-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.oil-price-table {
  width: 100%;
  margin-top: 15px;
  border-collapse: collapse;
  font-size: 12px; /* 减小字体大小 */
  white-space: nowrap; /* 防止文字换行 */
}

.oil-price-table th,
.oil-price-table td {
  border: 1px solid #ddd;
  padding: 6px; /* 减小内边距 */
  text-align: center;
}

.oil-price-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.oil-price-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.news-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px; /* 减小字体大小 */
  white-space: nowrap; /* 防止文字换行 */
}

.news-table th,
.news-table td {
  border: 1px solid #ddd;
  padding: 6px; /* 减小内边距 */
  text-align: center;
}

.news-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.news-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.news-title {
  color: #007bff;
  text-decoration: none;
}

.news-title:hover {
  text-decoration: underline;
}

.wiki-item {
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: rgba(232, 231, 231, 0.6);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.wiki-title {
  font-size: 18px;
  margin-bottom: 10px;
}

.wiki-content {
  font-size: 14px;
  line-height: 1.6;
  margin-bottom: 10px;
}

.wiki-link {
  color: #007bff;
  text-decoration: none;
}

.wiki-link:hover {
  text-decoration: underline;
}

</style>