<template>
  <el-card shadow="never">
    <el-row :gutter="10">
      <el-col :xs="24" :sm="24" :md="16" :lg="16">
          <div class="clearfix">
            <span style="line-height: 36px;">
              <i class="el-icon-share" style="margin-right: 4px;"></i>
              <a :href="leakageInfo.link" target="_blank">Suspicious item</a>
            </span>
            <a
              :href="'https://github.com/'+leakageInfo.project+'/search?utf8=✓&q=pass OR password OR passwd OR pwd OR smtp OR database'"
              target="_blank">
              <i class="el-icon-view" style="float: right;"></i>

            </a>
          </div>

          <div class="code-list" id="codelist">
            <div v-html="leakageInfo.detail"></div>
          </div>
      </el-col>
      <el-col :xs="24" :sm="24" :md="8" :lg="8">
        <div class="card-panel">
          <div style="padding: 15px;">
            <el-form :model="form">
              <el-form-item label="Is it safe?">
                <el-radio-group v-model="form.security">

                  <el-radio :label="1">Safe</el-radio>
                  <el-radio :label="0">Secret</el-radio>
                </el-radio-group>
              </el-form-item>
              <el-form-item label="Ignore repository">
                <el-radio-group v-model="form.ignore">

                  <el-radio :label="1">Ignored</el-radio>
                  <el-radio :label="0">Monitored</el-radio>
                </el-radio-group>
              </el-form-item>
              <el-form-item label="Comment">

                <el-input v-model="form.desc" type="textarea" placeholder="Please input ..."></el-input>

              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="dealLeakage(form)">Confirmed</el-button>
              </el-form-item>
            </el-form>
          </div>
        </div>
      </el-col>

      <el-col :span="24" v-if="code">

        <div class="card-panel">
          <el-collapse>
            <el-collapse-item title="受影响资产" name="affect">
              <el-table
                :data="affect"
                fit
                style="width: 100%">
                <el-table-column
                  prop="type"
                  label="泄露类型"
                >
                </el-table-column>
                <el-table-column
                  prop="value"
                  label="受影响资产"
                >
                </el-table-column>
              </el-table>
            </el-collapse-item>

          </el-collapse>
        </div>
        <div class="card-panel">
          <div class="clearfix">
            <span style="line-height: 36px;">
              <i class="el-icon-document" style="margin-right: 4px;"></i>
              <a :href="leakageInfo.link" target="_blank">可疑文件</a>
            </span>
          </div>
          <highlight-code :lang="leakageInfo.language">
            {{code}}
          </highlight-code>

        </div>
      </el-col>
    </el-row>

  </el-card>
</template>
<script>
  import {Base64} from 'js-base64';

  export default {
    data() {
      return {
        leakageInfo: {},
        code: '',
        affect: null,
        form: {security: 1, ignore: 1, desc: ''}
      }
    }, methods: {
      fetchInfoData() {
        this.axios.get(`${this.GLOBAL.leakage}/${this.$route.params.id}/info`)
          .then((response) => {
            this.leakageInfo = response.data.result[0];
            this.form.security = this.leakageInfo.security;
            this.form.project = this.leakageInfo.project;
            this.form.ignore = this.leakageInfo.ignore;
            if (this.leakageInfo.desc) {
              this.form.desc = Base64.decode(this.leakageInfo.desc);
            }
          })
          .catch((error) => {
            this.$message.error(error.toString());
          });
      },
      fetchCodeData() {
        this.axios.get(`${this.GLOBAL.leakage}/${this.$route.params.id}/code`)
          .then((response) => {
            this.code = Base64.decode(response.data.result[0].code);
            this.affect = response.data.affect;

          })
          .catch((error) => {
            this.$message.error(error.toString());
          });
      },
      dealLeakage(form) {
        this.form.id = this.$route.params.id;
        this.axios.patch(this.GLOBAL.leakage, this.form
        )
          .then((response) => {
            this.$message.success(response.data.msg);
          })
          .catch((error) => {
            this.$message.error(error.toString());

          });
      }
    },
    mounted() {
      this.fetchInfoData();
      this.fetchCodeData();
      this.$nextTick(function () {

      });
    }
  }
</script>

<style>
  @import "../style/github01.css";
  @import "../style/github02.css";
</style>
