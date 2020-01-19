<template>
  <div class="pr-20 pb-20">
    <div>
      <img src="@/static/icon/lookBoard.png" alt="" class="lookBoardLogo mr-7">
      <span>{{displayBoardMes.name}}</span>
    </div>

    <div class="uploadImg d-flex mt-30 fl position-relative">
      <el-upload
        :data="uploadData"
        :headers="uploadHeader"
        :show-file-list="false"
        :file-list="file_list"
        :on-success="reuploadSuccess"
        :on-progress="progress"
        :on-error="error"
        :before-upload="beforeUpload"
        v-if="installationId || isUploadPic"
        class="position-absolute top-1 right-2"
        style="z-index: 12;"
        action="/api/files/upload"
        accept=".jpeg, .jpg, .png">
        <el-button size="small" type="primary">{{$t('operation.reUpload')}}</el-button>
      </el-upload>

      <div
        v-cloak
        v-if="installationId || isUploadPic"
        :style="{ backgroundImage: `url(/files${factilitiesMes.picture})`}"
        class="installationShow d-flex ">
        <div @dragover="allowDrop($event)" @drop="drop($event)" class="imgDataPointContainer">
          <div
            v-cloak
            v-for="(item, index) in warningLightsData"
            :key="index"
            :style="[
              divStyle,{
                top: `${item.location.split(',')[1] * imgRadio}px`,
                left: `${parseInt(item.location.split(',')[0]) * imgRadio - warningLight.width * 0.65 / 2}px`
              }]"
            :class="{currentLight : index === focusWarningLights}">
            <img
              :draggable="false"
              @click="showDataPointFormGroup($event, index)"
              @contextmenu="showDeleteDataPoint($event, index)"
              :style="{width: warningLight.width + 'px', height: warningLight.height + 'px', zIndex: 2}"
              src="@/static/img/warningLights.png"
              alt=""
            >
            <input :style="inputStyle" :placeholder="$t('placeholder.enterName')" v-model="item.name"/>
          </div>
        </div>
        <div
          v-show="deleteDataPointShow"
          @click="deleteDataPoint($event)"
          class="deletePointMenu position-absolute"
          :style="{top: deletePointMenuTop + 'px', left: deletePointMenuLeft + 'px'}">
          <i class="el-icon-delete"></i>
          <span>{{ $t("newMonitoringFacilities.removeDataPoints") }}</span>
        </div>
      </div>

      <el-upload
        v-else
        :show-file-list="false"
        :data="uploadData"
        :headers="uploadHeader"
        :file-list="file_list"
        :before-upload="beforeUpload"
        :on-success="success"
        :on-progress="progress"
        :on-error="error"
        action="/api/files/upload"
        accept=".jpeg, .jpg, .png"
        class="upload d-flex"
      >
        <el-button size="small" type="primary">{{ $t("newMonitoringFacilities.uploadDeviceDiagram") }}</el-button>
        <div slot="tip" class="el-upload__tip">{{ $t("newMonitoringFacilities.uploadLimit") }}</div>
      </el-upload>
    </div>

    <template v-if="installationId || isUploadPic">
      <div class="rightContent d-flex mt-30">
        <div class="factilitiesName">
          <div v-show="!isEditName" class="text-20">
            <span>{{factilitiesMes.name ? factilitiesMes.name : '请输入名称' }}</span>
            <el-button type="text" class="ml-5" @click="editFacilityName">
              <i class="el-icon-edit"></i>
            </el-button>
          </div>
          <div v-show="isEditName" class="text-20">
            <el-input v-model="newFacilitiesName" ref="facilityNameInput"></el-input>
            <el-button type="text" class="ml-5" @click="saveNewName">
              <i class="el-icon-check"></i>
            </el-button>
            <el-button type="text" class="ml-5" @click="isEditName = false">
              <i class="el-icon-close"></i>
            </el-button>
          </div>
        </div>
        <img
          id="warningLight"
          @dragstart="drag($event)"
          src="@/static/img/warningLights.png"
          alt=""
          draggable="true"/>
        <ul>
          <li class="position-relative tipsPoint">{{ $t("newMonitoringFacilities.tipsPoint.first") }}</li>
          <li class="position-relative tipsPoint">{{ $t("newMonitoringFacilities.tipsPoint.second") }}</li>
          <li class="position-relative tipsPoint">{{ $t("newMonitoringFacilities.tipsPoint.third") }}</li>
        </ul>
      </div>
      <form-group-comp
        ref="formGroupComp"
        v-show="showFormGroup"
        :data="search"
        :key-factors="keyFactors"
        :show-key-factor="Boolean(search.formGroup)"
        :is-monitor-look-board="true"
        :key-factor-is-input="false"
        :able-search="Boolean(search.formGroup)"
        @formGroupCompSearch="searchSuccess"
        @formGroupKeyFactors="setKeyFactors"
        class="mt-30 mb-20"
        label-width="95px">
      </form-group-comp>

      <div v-show="queryDataShow">
        <el-table
          ref="queryDataTable"
          :data="allMonitorSetting"
          :header-cell-style="{background:'#0EA8D0',color:'#fff'}"
          @selection-change="selectChange"
          class="mt-40"
          stripe
        >
          <el-table-column
            type="selection">
          </el-table-column>
          <el-table-column
            prop="chartId"
            label="Chart ID">
          </el-table-column>
          <el-table-column
            :prop="`dataItem.labelGroup.factorPairs.${val.baseFactor.name}`"
            :label="val.baseFactor.description ? val.baseFactor.description : val.baseFactor.name"
            v-for="val in formGroupKeyFactorsMes"
            :key="val.id">
          </el-table-column>
          <el-table-column
            :prop="`factorPairs.${val.baseFactor.name}`"
            :label="val.baseFactor.description ? val.baseFactor.description : val.baseFactor.name"
            v-for="val in formGroupAttrFactorsMes"
            :key="val.id">
          </el-table-column>
        </el-table>

        <el-row class="mt-20">
          <el-col :span="24">
            <el-button
              @click="addToBePending"
              class="fr px-30"
              type="primary"
              plain>
              {{ $t("operation.addIn")}}
            </el-button>
          </el-col>
        </el-row>
      </div>

      <div class="formGroupDataTable mt-20 px-20 py-15" v-show="showFormGroup">
        <el-table :data="selectLightData">
          <el-table-column prop="formGroup.name" label="表单群组"></el-table-column>
          <el-table-column
            v-for="item in autoReportFactor"
            :key="item.id"
            :prop="`dataItem.labelGroup.factorPairs.${item.name}`"
            :label="item.description ? item.description : item.name"
          ></el-table-column>
          <el-table-column prop="chartId" label="Chart ID"></el-table-column>
          <el-table-column>
            <template slot-scope="scope">
              <el-button type="text">
                <i @click="deleteItem(scope.$index)" class="el-icon-delete"></i>
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <el-row class="mt-20">
        <el-col :span="24" style="text-align:center;">
          <el-button @click="saveFacilities" type="primary" class="px-30">{{ $t("operation.save") }}</el-button>
        </el-col>
      </el-row>
    </template>
    <div v-show="deleteDataPointShow" @click="deleteDataPointShow = false" class="mask"></div>
    <upload-progress v-show="!isAllowUpload" :percentage="percent"></upload-progress>
  </div>
</template>
<script>
import { mapState, mapActions } from 'vuex';
import UploadProgress from '@/components/uploadProgress.vue';
import formGroupComp from '@/components/formGroupComp.vue';
export default {
  name: '',
  components: {
    UploadProgress,
    formGroupComp
  },
  data() {
    return {
      search: {
        formGroup: ''
      },
      keyFactors: {},
      displayBoardMes: {}, // 当前监控看板的详细信息
      installationId: '', // 设施的id 新增时为new
      factilitiesMes: {}, // 设施的详细信息
      lookBoardTitle: '', // 监控看板名称
      reuploadDialog: false,
      dataPointTableShow: false,
      warningLightsData: [],
      file_list: [],
      formGroup: '',
      willDeletePoint: '', // 要删除的监控灯的index
      deleteDataPointShow: false,
      queryDataShow: false,
      installationImg: '',
      warningLight: '',
      focusWarningLights: '', // 选中的警示灯的index
      showFormGroup: false,
      imgRadio: 0, // 图片缩放比例
      isAllowUpload: true, // 允许上传
      labelGroupsId: [], // 表单群组搜索后返回的labelgroupid
      allMonitorSetting: [], // 所有labelGroupid对应的监控设定
      formGroupDetailMes: [], // 选择的表单群组的细节信息
      percent: 0, // 上传进度
      dragData: {
        pointerInImgX: '', // 拖拽时鼠标点击点距离图片水平距离
        pointerInImgY: '', //  拖拽时鼠标点击点距离图片垂直距离
        containerToImgX: '', // 拖拽结束时图片离容器的水平距离
        containerToImgY: '' // 拖拽结束时图片离容器的垂直距离
      }, // 拖拽中需要的数据
      // 图片上警示灯的外层div的样式
      allMonitorPoints: [],
      divStyle: {
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        position: 'absolute'
      },
      multipleSelection: [],
      selectLightData: [],
      isUploadPic: false,
      isEditName: false, // 是否在编辑设施名
      newFacilitiesName: '', // 新设施名
      uploadHeader: {}, // 上传的header
      uploadData: {}, // 上传的参数
      deletePointMenuTop: 0,
      deletePointMenuLeft: 0
    };
  },
  computed: {
    ...mapState({
      displayBoards: state => state.lookBoardSetting.displayBoards,
      allFactors: state => state.factor.allFactors
    }),
    autoReportFactor() { // 自动上报因子
      return this.allFactors.filter((val) => {
        return val.isAutoReportNecessary;
      });
    },
    // 图片上警示灯的下input的样式
    inputStyle() {
      return {
        width: this.warningLight.width * 1.65 + 'px',
        border: 'none',
        textAlign: 'center',
        background: '#fff',
        fontSize: '12px',
        marginTop: '6px',
        zIndex: 2
      };
    },
    // 根据formGroupDetailMes拿出关键因子信息
    formGroupKeyFactorsMes() {
      return this.formGroupDetailMes.filter((val) => {
        return val.baseFactor.isAutoReportNecessary;
      });
    },
    // 根据formGroupDetailMes拿出非关键因子信息
    formGroupAttrFactorsMes() {
      return this.formGroupDetailMes.filter((val) => {
        return !val.baseFactor.isAutoReportNecessary;
      });
    }
  },
  watch: {
    'search.formGroup'(newVal) {
      if (newVal !== '') {
        this.getFormGroupDetail(newVal).then((res) => {
          this.formGroupDetailMes = res.formGroupFactors;
        });
      }
    }
  },
  created() {
    this.installationId = this.$route.params.facilityId;
    if (this.installationId) {
      this.queryFacilityById(parseInt(this.installationId)).then((res) => {
        this.factilitiesMes = res.data;
        this.displayBoardMes = res.data.displayBoard;
        if (res.data.monitorPoints) {
          this.warningLightsData = res.data.monitorPoints;
        }
        this.$nextTick(() => {
          this.initDataPoint();
        });
      });
    } else {
      const displayBoardId = this.$route.params.lookBoardId;
      this.queryDisplayBoardsById(parseInt(displayBoardId)).then((res) => {
        this.displayBoardMes = res.data;
      });
    }
  },
  mounted() {
    this.uploadData = { fileType: 'IMGS' };
    this.uploadHeader.Authorization = this.$cookies.get('token');
  },
  methods: {
    ...mapActions([
      'queryDisplayBoardsById',
      'queryFacilityById',
      'getAllProjectByCondition',
      'addNewFacilities',
      'putFacilities',
      'getFormGroupDetail'
    ]),
    drag(ev) {
      ev.stopPropagation();
      ev.dataTransfer.setData('URI', ev.target.currentSrc);
      this.dragData.pointerInImgX = ev.offsetX;
      this.dragData.pointerInImgY = ev.offsetY;
    },
    allowDrop(ev) {
      ev.stopPropagation();
      ev.preventDefault();
    },
    drop(ev) {
      if (ev.target.className !== 'imgDataPointContainer') {
        return;
      }
      ev.stopPropagation();
      ev.preventDefault();
      this.dragData.containerToImgX = ev.layerX;
      this.dragData.containerToImgY = ev.layerY;
      const layerX = this.dragData.containerToImgX - this.dragData.pointerInImgX;
      const layerY = this.dragData.containerToImgY - this.dragData.pointerInImgY;
      this.warningLightsData.push({
        name: '',
        location: `${layerX / this.imgRadio},${layerY / this.imgRadio}`,
        description: '',
        monitorSettings: []
      });
    },
    beforeUpload(file) { // 上传
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onloadend = function(e) {
        console.log(e);
      };
      const fileType = file.name.split('.')[1];
      const fileSize = file.size / 1024 / 1024;
      let canUpload = false;
      if ((fileType === 'jpg' || fileType === 'jpeg' || fileType === 'png') && fileSize < 10) {
        canUpload = true;
      }
      if (!canUpload) {
        this.isAllowUpload = false;
        return false;
      } else {
        return true;
      }
    },
    // 文件上传中
    progress(ev) {
      this.percent = Math.round(ev.percent);
    },
    // 上传成功
    success(res) {
      this.isAllowUpload = true;
      this.percent = 0;
      this.factilitiesMes.picture = res;
      this.initDataPoint();
      this.isUploadPic = true;
    },
    reuploadSuccess(res) {
      this.isAllowUpload = true;
      this.percent = 0;
      this.factilitiesMes.picture = res;
      this.$set(this, 'warningLightsData', []);
      this.showFormGroup = false;
      this.queryDataShow = false;
      this.search.formGroup = '';
      this.keyFactors = {};
      this.allMonitorSetting = [];
      this.selectLightData = [];
      this.initDataPoint();
    },
    // 上传失败
    error() {
      this.isAllowUpload = true;
      this.percent = 0;
    },
    editFacilityName() {
      this.newFacilitiesName = this.factilitiesMes.name;
      this.isEditName = true;
      this.$nextTick(() => {
        this.$refs.facilityNameInput.focus();
      });
    },
    // 保存设施名  失焦时触发
    saveNewName() {
      this.isEditName = false;
      this.factilitiesMes.name = this.newFacilitiesName;
    },
    loadingPicture(url) { // 图片加载
      return new Promise(function(resolve, reject) {
        const image = new Image();
        image.src = url;
        if (url === '') {
          reject(new Error(this.$t('newMonitoringFacilities.imgNullErr')));
        }
        image.onload = function() {
          resolve(this);
        };
      });
    },
    // 初始化图片数据
    async initDataPoint() { // 计算图片缩放比例 绘制警示灯
      const _this = this;
      try {
        // 加载图片
        const img = await this.loadingPicture(`/files${_this.factilitiesMes.picture}`);
        const imgContainer = document.getElementsByClassName('installationShow')[0];
        // 获取图片缩放比例
        const heightRatio = imgContainer.offsetHeight / img.height;
        const autoWidth = img.width * heightRatio;
        if (autoWidth > imgContainer.offsetWidth) {
          _this.imgRadio = imgContainer.offsetWidth / img.width;
        } else {
          _this.imgRadio = heightRatio;
        }
        const dataPointContainer = document.getElementsByClassName('imgDataPointContainer')[0];
        const radioWidth = img.width * _this.imgRadio;
        const radioHeight = img.height * _this.imgRadio;
        dataPointContainer.style.width = radioWidth + 'px';
        dataPointContainer.style.height = radioHeight + 'px';
        dataPointContainer.style.top = (imgContainer.offsetHeight - radioHeight) / 2 + 'px';
        dataPointContainer.style.left = (imgContainer.offsetWidth - radioWidth) / 2 + 'px';
        const restrictedRadio = 0.6;
        _this.warningLight = await this.loadingPicture('/img/warningLights.png');
        _this.warningLight.width *= restrictedRadio;
        _this.warningLight.height *= restrictedRadio;
        // 提示上面的灯按比例缩放
        const tipsWarningLight = document.getElementById('warningLight');
        tipsWarningLight.width = _this.warningLight.width;
        tipsWarningLight.height = _this.warningLight.height;
      } catch (ignore) {}
    },
    showDeleteDataPoint(ev, index) { // 显示数据点删除框
      // 转数字
      const targetImgTop = +ev.target.parentElement.style.top.slice(0, -2);
      // 转数字
      const targetImgLeft = +ev.target.parentElement.style.left.slice(0, -2);
      // 转数字
      const targetContaTop = +ev.target.parentElement.parentElement.style.top.slice(0, -2);
      // 转数字
      const targetContaLeft = +ev.target.parentElement.parentElement.style.left.slice(0, -2);
      // img 和父 div 的距离
      const imgDivWidth = (ev.target.parentElement.offsetWidth - ev.target.offsetWidth) / 2;
      this.deleteDataPointShow = true;
      this.deletePointMenuTop = targetContaTop + targetImgTop + ev.offsetY;
      this.deletePointMenuLeft = targetContaLeft + targetImgLeft + ev.offsetX + imgDivWidth;
      this.willDeletePoint = index;
    },
    deleteDataPoint(ev) { // 删除数据点
      this.$confirm(this.$t('newMonitoringFacilities.confirmPrompt'), this.$t('message.tips'), {
        confirmButtonText: this.$t('operation.confirm'),
        cancelButtonText: this.$t('operation.cancel'),
        type: 'warning'
      }).then(() => {
        this.warningLightsData.splice(this.willDeletePoint, 1);
        this.deleteDataPointShow = false;
        if (this.willDeletePoint === this.focusWarningLights) {
          this.showFormGroup = false;
          this.queryDataShow = false;
        }
      });
    },
    // 表单群组搜索
    searchSuccess(data) {
      data = data.join(',');
      this.getAllProjectByCondition({ labelGroupIds: data }).then((res) => {
        this.allMonitorSetting = res.data;
        this.queryDataShow = true;
      });
    },
    showDataPointFormGroup(ev, index) { // 点击警示灯显示表格
      this.queryDataShow = false;
      this.search.formGroup = '';
      this.showFormGroup = true;
      this.focusWarningLights = index;
      this.selectLightData = this.warningLightsData[index].monitorSettings;
      this.allMonitorSetting = [];
      // if (this.warningLightsData[index].monitorSettings.length !== 0) {
      //   this.search.formGroup = this.warningLightsData[index].monitorSettings[0].formGroup.id;
      //   this.$refs.formGroupComp.selectedFormGroup(this.search.formGroup);
      // }
    },
    selectChange(val) { // 选中表格中的数据
      this.multipleSelection = val;
    },
    addToBePending() { // 将table中的数据加入待存区
      const res = new Map();
      const allSelection = [].concat(this.selectLightData, this.multipleSelection);
      this.selectLightData = allSelection.filter((val) => {
        return !res.has(val.chartId) && res.set(val.chartId, true);
      });
      this.warningLightsData[this.focusWarningLights].monitorSettings = this.selectLightData;
    },
    deleteItem(index) {
      this.selectLightData.splice(index, 1);
      this.warningLightsData[this.focusWarningLights].monitorSettings.splice(index, 1);
    },
    // 设置表单群组的关键因子
    setKeyFactors(keyFactors) {
      this.$set(this, 'keyFactors', {
        ...keyFactors
      });
    },
    saveFacilities() {
      // for (let i = 0; i < this.warningLightsData.length; i++) {
      //   if (this.warningLightsData[i].name === '') {
      //     this.$message.warning('请输入警示灯的名字。');
      //     return false;
      //   }
      // }
      if (!this.factilitiesMes.name) {
        this.$message.warning('请输入设施的名字。');
        return false;
      }
      for (let i = 0; i < this.warningLightsData.length; i++) {
        this.warningLightsData[i].monitorSettings = this.warningLightsData[i].monitorSettings.map((val) => {
          return {
            id: val.id
          };
        });
      }
      const req = {
        name: this.factilitiesMes.name ? this.factilitiesMes.name : '',
        description: '',
        picture: this.factilitiesMes.picture,
        displayBoard: {
          id: this.displayBoardMes.id
        },
        monitorPoints: this.warningLightsData
      };
      if (!this.installationId) {
        this.addNewFacilities(req).then(() => {
          this.$message.success(this.$t('message.successSave'));
          this.$router.push({ path: this.$i18n.path('systemDesign/monitorDesign/lookBoardDesign') });
        });
      } else {
        req.id = this.installationId;
        this.putFacilities(req).then(() => {
          this.$message.success(this.$t('message.successEdit'));
          this.$router.push({ path: this.$i18n.path('systemDesign/monitorDesign/lookBoardDesign') });
        });
      }
    }
  }
};
</script>
<style lang="scss" scoped>
  .lookBoardLogo{
    width: 22px;
    height: 20px;
  }
  .uploadImg{
    width:682px;
    height:462px;
    background:rgba(255,255,255,1);
    border:1px dashed rgba(14,168,208,1);
    border-radius:10px;
    justify-content: center;
    align-items: center;
    .upload{
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }
  }
  .tipsPoint{
    margin-top: 20px;
    font-size: 16px;
    max-width: 400px;
    &::before{
      content: '';
      display: inline-block;
      width:6px;
      height:6px;
      background:rgba(14,168,208,1);
      border-radius:50%;
      position: absolute;
      top:50%;
      left:-15px;
      transform: translateY(-50%);
    }
  }
  .rightContent{
    position: relative;
    margin-left:682px;
    min-width: 360px;
    height:462px;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .factilitiesName{
    position: absolute;
    top:5px;
    left:20px;
    font-size: 0;
    display: inline-block;
    white-space: pre;
    /deep/.el-button{
      font-size: 20px;
    }
  }
  .installationShow{
    width:100%;
    height: 100%;
    position: relative;
    justify-content: center;
    align-items: center;
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
  }
  .deletePointMenu{
    min-width:118px;
    height:40px;
    padding:0 15px;
    background:rgba(255,255,255,1);
    box-shadow:0px 0px 9px 0px rgba(106,106,106,0.26);
    text-align: center;
    line-height: 40px;
    cursor: pointer;
    z-index: 10;
  }
  .mask{
    position: absolute;
    top:0;
    left:0;
    right:0;
    bottom: 0;
    background: transparent;
    z-index: 9;
  }
  .imgDataPointContainer{
    position: absolute;
  }
  .currentLight::after{
    content: '';
    width: 54px;
    height: 54px;
    background: rgb(215, 244, 252);
    border: 1px solid rgb(29, 173, 212);
    top: -4px;
    left: 50%;
    transform: translateX(-50%);
    border-radius: 50%;
    position: absolute;
    z-index: 1;
  }
</style>
