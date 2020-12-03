<template>
  <div class="wrapper">
    <div class="grid-one">
      <div class="redis_cluster_select">
        <div class="name_select">
          <el-tag>redis集群选择（cluster_name)</el-tag>
          <el-select
            v-model="value"
            :remote-method="remoteSearchRedisClusterName"
            :loading="loading"
            size="medium"
            filterable
            remote
            clearable
            reserve-keyword
            default-first-option
            placeholder="redis集群名关键词选择"
            style="width: 60%"
            @change="setSelectedServerIdName">
            <el-option
              v-for="item in cluster_name_options"
              :key="item.label"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </div>
        <div class="host_select">
          <el-tag>redis集群选择（hosts)</el-tag>
          <el-select
            v-model="value"
            :remote-method="remoteSearchRedisClusterHosts"
            :loading="loading"
            size="medium"
            filterable
            remote
            clearable
            reserve-keyword
            default-first-option
            placeholder="redis集群hosts关键词选择"
            style="width: 70%"
            @change="setSelectedServerIdHosts">
            <el-option
              v-for="item in cluster_hosts_options"
              :key="item.label"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </div>
      </div>
    </div>
    <div class="grid-two">
      <el-tag style="width: 90%; text-align: center">结果输出</el-tag>
      <div class="reslut" >
        <pre>{{ result_code }}<br>{{ result_msg }}</pre>
      </div>
    </div>
    <div class="grid-three">
      <div class="search_all_key">
        <el-button type="primary" @click="handlershowALLKey">查看所有key</el-button>
        <el-button type="primary" @click="handlershowDBSize">查看key数量(dbsize)</el-button>
      </div>
    </div>
    <div class="grid-four">
      <div class="get_key_value">
        <el-collapse>
          <el-collapse-item title="查询键值" name="1">
            <el-input
              v-model="input_key_for_value"
              placeholder="请输入要查询的键(key)"
              clearable
              style="width: 40%">
            </el-input>
            <el-button type="primary" plain @click="handleGetKey">开始查询</el-button>
          </el-collapse-item>
          <el-collapse-item title="TTL查询" name="2">
            <el-input
              v-model="input_key_for_ttl"
              placeholder="请输入要查询的键(key)"
              clearable
              style="width: 40%">
            </el-input>
            <el-button type="primary" plain @click="handleGetKeyTTL">开始查询</el-button>
          </el-collapse-item>
          <el-collapse-item title="删除键-值(key-value)" name="3">
            <el-input
              v-model="input_key_for_del"
              placeholder="请输入要删除的键(key)"
              clearable
              style="width: 40%">
            </el-input>
            <el-popconfirm
              confirm-button-text="好的"
              cancel-button-text="不用了"
              icon="el-icon-info"
              icon-color="red"
              title="确定删除吗？"
              @confirm="handleDelKey"
            >
              <el-button slot="reference" type="danger">删除</el-button>
            </el-popconfirm>
          </el-collapse-item>
          <el-collapse-item title="新建键-值(key-value)" name="4">
            <el-input
              v-model="input_key_for_create"
              placeholder="请输入要新建的键(key)"
              clearable
              style="width: 40%">
            </el-input>
            <el-input
              v-model="input_value_for_create"
              placeholder="请输入要新建的值(value)"
              clearable
              style="width: 40%">
            </el-input>
            <el-button type="primary" plain @click="handleCreateKey">新建</el-button>
          </el-collapse-item>
        </el-collapse>
      </div>
    </div>
  </div>
</template>

<script>
import { getServerList, showALLKey, showDBSize, getKey, getKeyTTL, delKey, createKey } from '@/api/redis_server'
// import { getSupplierList } from '@/api/supplier'
// import { getManufactoryList } from '@/api/manufactory'
// import { getIdcList } from '@/api/idc'

export default {
  name: 'Server',
  data() {
    return {
      cluster_name_options: [],
      cluster_hosts_options: [],
      value: this.$store.state.server_id,
      cluster_name_list: [],
      cluster_hosts_list: [],
      loading: false,
      result_code: '',
      result_msg: '',
      totalNum: 0,
      server: [],
      input_key_for_value: '',
      input_key_for_ttl: '',
      input_key_for_del: '',
      input_key_for_create: '',
      input_value_for_create: '',
      selected_server_id: null,
      params: {
        page: 1,
        keywords: '',
        page_size: 10
      },
      kwargs: {}
    }
  },
  created() {
    this.fetchData()
  },
  // mounted() {
  // },
  methods: {
    fetchData() {
      getServerList(this.params).then(
        // 获取redis集群列表
        res => {
          this.server = res.results
          console.log('fetchdata')
          // console.log(this.server)
          this.totalNum = res.count
          //
          this.cluster_hosts_list = this.server.map(item => {
            return { value: `${item.id}`, label: `${item.hosts}` }
          })
          this.cluster_name_list = this.server.map(item => {
            return { value: `${item.id}`, label: `${item.cluster_name}` }
          })
          // console.log(this.cluster_name_list)
          if (this.$store.state.redis.server_id !== 0) {
            this.value = this.$store.state.redis.server_id
            // this.cluster_name_options = this.cluster_name_list.find(item => {
            //   if (item.value === this.value.toString()) {
            //     return item
            //   }
            // })
            // console.log(this.cluster_name_options)
            // this.cluster_hosts_options = this.cluster_hosts_list.find(item => {
            //   if (item.value === this.value.toString()) {
            //     return item
            //   }
            // })
            this.setSelectedServerIdName(this.value)
            this.setSelectedServerIdName(this.value)
            // console.log(this.cluster_hosts_options)
            // console.log(this.cluster_name_options)
          }
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    },
    remoteSearchRedisClusterName(query) {
      this.cluster_name_list = this.server.map(item => {
        return { value: `${item.id}`, label: `${item.cluster_name}` }
      })
      if (query !== '') {
        this.loading = true
        setTimeout(() => {
          this.loading = false
          this.cluster_name_options = this.cluster_name_list.filter(item => {
            return item.label.toLowerCase()
              .indexOf(query.toLowerCase()) > -1
          })
        }, 200)
      } else {
        this.cluster_name_options = []
      }
    },
    remoteSearchRedisClusterHosts(query) {
      this.cluster_hosts_list = this.server.map(item => {
        return { value: `${item.id}`, label: `${item.hosts}` }
      })
      if (query !== '') {
        this.loading = true
        setTimeout(() => {
          this.loading = false
          this.cluster_hosts_options = this.cluster_hosts_list.filter(item => {
            return item.label.toLowerCase()
              .indexOf(query.toLowerCase()) > -1
          })
        }, 200)
      } else {
        this.cluster_hosts_options = []
      }
    },
    setSelectedServerIdName(item) {
      this.selected_server_id = item
      this.cluster_hosts_options = this.server.map(i => {
        return { value: `${i.id}`, label: `${i.hosts}` }
      })
    },
    setSelectedServerIdHosts(item) {
      this.selected_server_id = item
      this.cluster_name_options = this.server.map(i => {
        return { value: `${i.id}`, label: `${i.cluster_name}` }
      })
    },
    handlershowALLKey() {
      showALLKey(this.value).then(
        // 获取redis集群列表
        res => {
          let str_key = '\n'
          for (var i in res.msg) {
            str_key = str_key + res.msg[i] + '\n'
          }
          this.result_code = '返回code：' + res.code
          this.result_msg = '返回结果：' + str_key
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    },
    handlershowDBSize() {
      showDBSize(this.value).then(
        // 获取redis集群列表
        res => {
          this.result_code = '返回code：' + res.code
          this.result_msg = '返回结果：' + res.msg
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    },
    handleGetKey() {
      this.kwargs.id = this.value
      this.kwargs.key = this.input_key_for_value
      getKey(this.kwargs).then(
        // 获取redis集群列表
        res => {
          this.result_code = '返回code：' + res.code
          this.result_msg = '返回结果：' + res.msg
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    },
    handleGetKeyTTL() {
      this.kwargs.id = this.value
      this.kwargs.key = this.input_key_for_ttl
      getKeyTTL(this.kwargs).then(
        // 获取redis集群列表
        res => {
          this.result_code = '返回code：' + res.code
          this.result_msg = '返回结果：' + res.msg
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    },
    handleDelKey() {
      this.kwargs.id = this.value
      this.kwargs.key = this.input_key_for_del
      delKey(this.kwargs).then(
        // 获取redis集群列表
        res => {
          this.result_code = '返回code：' + res.code
          this.result_msg = '返回结果：' + res.msg
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    },
    handleCreateKey() {
      this.kwargs.id = this.value
      this.kwargs.key = this.input_key_for_create
      this.kwargs.value = this.input_value_for_create
      createKey(this.kwargs).then(
        // 获取redis集群列表
        res => {
          console.log(res)
          this.result_code = '返回结果：' + res
          this.result_msg = ''
        },
        err => {
          this.$message({
            type: 'error',
            message: err.response.data.detail
          })
        }
      )
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .wrapper {
    display: grid;
    grid-template-columns: 30% 30% 40%;
    grid-template-rows: 30% 10% 40% 60%;
  }
  .grid-one {
    grid-column: 1 / 3;
    grid-row: 1 / 2;
  }
  .grid-two {
    grid-column: 3 / 4;
    grid-row: 1 / 5;
  }
  .grid-three {
    grid-column: 1 / 2;
    grid-row:  2 / 3 ;
  }
  .grid-four {
    grid-column: 1 / 3;
    grid-row: 3/ 4;
  }
  .redis_cluster_select {
    padding: 10px;
    margin-top: 2px;
  }
  .name_select {
    margin-top: 10px;
  }
  .host_select {
  /*padding: 10px;*/
    margin-top: 10px;
  }
  .search_all_key {
    margin-left: 20px;
    padding: 5px;
  }
  .get_key_value {
    margin-left: 20px;
    margin-top: 10px;
  }
  .reslut {
    background: darkgrey;
    width: 90%;
    height: 120%;
    margin: 0px;
    padding: 0px;
  }
  pre {
    margin: 0px;
    height: auto;
    overflow: auto;
    background-color: darkgrey;
    word-break: normal !important;
    word-wrap: normal !important;
    white-space: pre !important;
  }
</style>
