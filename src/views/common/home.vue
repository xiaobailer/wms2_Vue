<!--
<template>
  <div class="mod-home">
    <h3>项目介绍</h3>
    <ul>
      <li>renren-fast-vue基于vue、element-ui构建开发，实现<a href="https://gitee.com/renrenio/renren-fast" target="_blank">renren-fast</a>后台管理前端功能，提供一套更优的前端解决方案</li>
      <li>前后端分离，通过token进行数据交互，可独立部署</li>
      <li>主题定制，通过scss变量统一一站式定制</li>
      <li>动态菜单，通过菜单管理统一管理访问路由</li>
      <li>数据切换，通过mock配置对接口数据／mock模拟数据进行切换</li>
      <li>发布时，可动态配置CDN静态资源／切换新旧版本</li>
      <li>演示地址：<a href="http://demo.open.renren.io/renren-fast" target="_blank">http://demo.open.renren.io/renren-fast</a> (账号密码：admin/admin)</li>
    </ul>
  </div>
</template>

<script>
  export default {
  }
</script>

<style>
  .mod-home {
    line-height: 1.5;
  }
</style>
-->



<template>
  <div class="home-container">
    <h2 class="home-title">仓库管理系统</h2>
    <p class="home-description">提高物流效率，优化供应链管理</p>
    <div class="home-card-container">
      <div class="home-card" @click="$router.push('/inbound-wms2products')">
        <div class="home-card-header" > 
          <i class="el-icon-s-management"></i>
          <span>入库模块</span>
        </div>
        <div class="home-card-content">
          <p>包括商品信息、供应商信息，入库等流程。</p>
        </div>
      </div>
      <div class="home-card" @click="$router.push('/outbound-wms2outbound')">
        <div class="home-card-header">
          <i class="el-icon-s-finance"></i>
          <span>出库模块</span>
        </div>
        <div class="home-card-content">
          <p>包括出库单、出库审核管理等。</p>
        </div>
      </div>
      <div class="home-card" @click="$router.push('/warehouse-warehouse')">
        <div class="home-card-header">
          <i class="el-icon-s-order"></i>
          <span>仓库管理</span>
        </div>
        <div class="home-card-content">
          <p>仓库详细，预警等</p>
        </div>
      </div>
       <div class="home-card" @click="$router.push('/customer-wms2customer')">
        <div class="home-card-header">
          <i class="el-icon-s-order"></i>
          <span>客户模块</span>
        </div>
        <div class="home-card-content">
          <p>客户信息等</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
  }
</script>

<style scoped>
.home-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 60px 0;
}

.home-title {
  font-size: 32px;
  font-weight: bold;
  margin-bottom: 20px;
  color: #333;
}

.home-description {
  font-size: 16px;
  margin-bottom: 40px;
  color: #666;
}

.home-card-container {
  display: flex;
  justify-content: center;
  gap: 40px;
}

.home-card {
  width: 200px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: box-shadow 0.3s ease-in-out;
}

.home-card:hover {
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
}

.home-card-header {
  display: flex;
  align-items: center;
  padding: 20px;
  background-color: #f5f5f5;
}

.home-card-header i {
  font-size: 24px;
  margin-right: 10px;
  color: #409EFF;
}

.home-card-header span {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.home-card-content {
  padding: 20px;
  font-size: 14px;
  color: #666;
}
</style>
