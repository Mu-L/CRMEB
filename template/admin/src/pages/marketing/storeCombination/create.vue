<template>
  <div>
    <pages-header
      ref="pageHeader"
      :title="$route.params.id ? '编辑拼团商品' : '添加拼团商品'"
      :backUrl="$routeProStr + '/marketing/store_combination/index'"
    ></pages-header>
    <el-card :bordered="false" shadow="never" class="mt16">
      <el-row class="mt30 acea-row row-middle row-center">
        <el-col :span="20">
          <steps :stepList="stepList" :isActive="current"></steps>
        </el-col>
        <el-col :span="23" v-loading="spinShow">
          <el-form
            class="form mt30"
            ref="formValidate"
            :model="formValidate"
            :rules="ruleValidate"
            @on-validate="validate"
            :label-width="labelWidth"
            :label-position="labelPosition"
            @submit.native.prevent
          >
            <el-form-item label="选择商品：" prop="image_input" v-if="current === 0">
              <div class="picBox" v-db-click @click="changeGoods">
                <div class="pictrue" v-if="formValidate.image">
                  <img v-lazy="formValidate.image" />
                </div>
                <div class="upLoad acea-row row-center-wrapper" v-else>
                  <i class="el-icon-goods" style="font-size: 24px"></i>
                </div>
              </div>
            </el-form-item>
            <el-row v-show="current === 1">
              <el-col :span="24">
                <el-form-item label="商品主图：" prop="image">
                  <div class="picBox" v-db-click @click="modalPicTap('dan', 'danFrom')">
                    <div class="pictrue" v-if="formValidate.image">
                      <img v-lazy="formValidate.image" />
                    </div>
                    <div class="upLoad acea-row row-center-wrapper" v-else>
                      <i class="el-icon-picture-outline" style="font-size: 24px"></i>
                    </div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="商品轮播图：" prop="images">
                  <div class="acea-row">
                    <div
                      class="pictrue"
                      v-for="(item, index) in formValidate.images"
                      :key="index"
                      draggable="true"
                      @dragstart="handleDragStart($event, item)"
                      @dragover.prevent="handleDragOver($event, item)"
                      @dragenter="handleDragEnter($event, item)"
                      @dragend="handleDragEnd($event, item)"
                    >
                      <img v-lazy="item" />
                      <i class="el-icon-circle-close btndel" v-db-click @click="handleRemove(index)"></i>
                    </div>
                    <div
                      v-if="formValidate.images.length < 10"
                      class="upLoad acea-row row-center-wrapper"
                      v-db-click
                      @click="modalPicTap('duo')"
                    >
                      <i class="el-icon-picture-outline" style="font-size: 24px"></i>
                    </div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-col v-bind="grid">
                  <el-form-item label="拼团名称：" prop="title" label-for="title">
                    <el-input
                      elearable
                      placeholder="请输入拼团名称"
                      v-model="formValidate.title"
                      class="content_width"
                      maxlength="80"
                      show-word-limit
                    />
                  </el-form-item>
                </el-col>
              </el-col>
              <el-col :span="24">
                <el-col v-bind="grid">
                  <el-form-item label="拼团简介：" prop="info" label-for="info">
                    <el-input
                      placeholder="请输入拼团简介"
                      type="textarea"
                      :rows="4"
                      v-model="formValidate.info"
                      class="content_width"
                      maxlength="100"
                      show-word-limit
                    />
                  </el-form-item>
                </el-col>
              </el-col>
              <el-col :span="24">
                <el-form-item label="拼团时间：" prop="section_time">
                  <div>
                    <el-date-picker
                      clearable
                      :editable="false"
                      type="datetimerange"
                      format="yyyy-MM-dd HH:mm"
                      value-format="yyyy-MM-dd HH:mm"
                      range-separator="-"
                      start-placeholder="开始日期"
                      end-placeholder="结束日期"
                      @change="onchangeTime"
                      class="content_width"
                      v-model="formValidate.section_time"
                    ></el-date-picker>
                    <div class="grey">设置活动开启结束时间，用户可以在设置时间内发起参与拼团</div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24" v-if="formValidate.virtual_type == 0">
                <el-form-item label="物流方式：" prop="logistics">
                  <el-checkbox-group v-model="formValidate.logistics">
                    <el-checkbox label="1">快递</el-checkbox>
                    <el-checkbox label="2">到店</el-checkbox>
                  </el-checkbox-group>
                </el-form-item>
              </el-col>
              <el-col :span="24" v-if="formValidate.virtual_type == 0">
                <el-form-item label="运费设置：" :prop="formValidate.freight != 1 ? 'freight' : ''">
                  <el-radio-group v-model="formValidate.freight">
                    <el-radio :label="2">固定邮费</el-radio>
                    <el-radio :label="3">运费模板</el-radio>
                  </el-radio-group>
                </el-form-item>
              </el-col>
              <el-col
                :span="24"
                v-if="formValidate.freight != 3 && formValidate.freight != 1 && formValidate.virtual_type == 0"
              >
                <el-form-item label="">
                  <div class="acea-row">
                    <el-input-number
                      :controls="false"
                      :min="0"
                      :max="9999999999"
                      v-model="formValidate.postage"
                      placeholder="请输入金额"
                      class="content_width input-number-unit-class"
                      class-unit="元"
                    />
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24" v-if="formValidate.freight == 3 && formValidate.virtual_type == 0">
                <el-form-item label="" prop="temp_id">
                  <div class="acea-row">
                    <el-select
                      v-model="formValidate.temp_id"
                      clearable
                      placeholder="请选择运费模板"
                      class="content_width"
                    >
                      <el-option
                        v-for="(item, index) in templateList"
                        :value="item.id"
                        :key="index"
                        :label="item.name"
                      ></el-option>
                    </el-select>
                    <span class="addfont" v-db-click @click="freight">新增运费模板</span>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="拼团时效：" prop="effective_time">
                  <div>
                    <el-input-number
                      :controls="false"
                      placeholder="请输入拼团时效"
                      class="content_width input-number-unit-class"
                      class-unit="小时"
                      v-model="formValidate.effective_time"
                    />
                    <div class="grey">
                      用户发起拼团后开始计时，需在设置时间内邀请到规定好友人数参团，超过时效时间，则系统判定拼团失败，自动发起退款
                    </div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="拼团人数：" prop="people">
                  <div>
                    <el-input-number
                      :controls="false"
                      :min="2"
                      :max="10000"
                      placeholder="请输入拼团人数"
                      :precision="0"
                      v-model="formValidate.people"
                      class="content_width input-number-unit-class"
                      class-unit="人"
                    />
                    <div class="grey">单次拼团需要参与的用户数</div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="虚拟成团补齐人数：" prop="virtualPeople">
                  <div>
                    <el-input-number
                      :controls="false"
                      placeholder="设置虚拟成团的补齐人数"
                      :precision="0"
                      :max="10000"
                      :min="0"
                      v-model="formValidate.virtualPeople"
                      class="content_width input-number-unit-class"
                      class-unit="人"
                    />
                    <div class="grey">
                      设置虚拟成团的补齐人数，如：5人团设置补齐2人，当团队成员大于等于3人时，拼团结束时自动补齐剩余最多2个位置，不开启虚拟成团请设置为0
                    </div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="单位：" prop="unit_name" label-for="unit_name">
                  <el-input clearable placeholder="请输入单位" v-model="formValidate.unit_name" class="content_width" />
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="总购买数量限制：" prop="num">
                  <div>
                    <el-input-number
                      :controls="false"
                      :min="1"
                      placeholder="请输入总数量限制"
                      :precision="0"
                      :max="10000"
                      v-model="formValidate.num"
                      class="content_width input-number-unit-class"
                      :class-unit="formValidate.unit_name || '件'"
                    />
                    <div class="grey">
                      该商品活动期间内，用户可购买的最大数量。例如设置为4，表示本次活动有效期内，每个用户最多可购买4件
                    </div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="单次购买数量限制：" prop="once_num">
                  <div>
                    <el-input-number
                      :controls="false"
                      :min="1"
                      placeholder="请输入单次购买数量限制"
                      :precision="0"
                      :max="10000"
                      v-model="formValidate.once_num"
                      class="content_width input-number-unit-class"
                      :class-unit="formValidate.unit_name || '件'"
                    />
                    <div class="grey">
                      用户参与拼团时，一次购买最大数量限制。例如设置为2，表示每次参与拼团时，用户一次购买数量最大可选择2个
                    </div>
                  </div>
                </el-form-item>
              </el-col>

              <el-col :span="24">
                <el-form-item label="团长返佣比例：" prop="head_commission">
                  <div>
                    <el-input-number
                      :controls="false"
                      :min="0"
                      :max="100"
                      placeholder="团长返佣比例"
                      :precision="0"
                      v-model="formValidate.head_commission"
                      class="content_width input-number-unit-class"
                      class-unit="%"
                    />
                    <div class="grey">
                      拼团成功后，如果团长是分销员，则在订单确认收货时会给团长返一定的佣金，佣金比例是实际支付金额的0-100%
                    </div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="拼团是否参与分销：" props="is_commission" label-for="is_commission">
                  <div>
                    <el-switch
                      class="defineSwitch"
                      :active-value="1"
                      :inactive-value="0"
                      v-model="formValidate.is_commission"
                      size="large"
                      active-text="开启"
                      inactive-text="关闭"
                    >
                    </el-switch>
                    <div class="grey">拼团商品是否参与商城分销返佣</div>
                  </div>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="排序：">
                  <el-input-number
                    :controls="false"
                    placeholder="请输入排序"
                    :precision="0"
                    :max="10000"
                    :min="0"
                    v-model="formValidate.sort"
                    class="content_width"
                  />
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="热门推荐：" props="is_hot" label-for="is_hot">
                  <el-switch
                    class="defineSwitch"
                    :active-value="1"
                    :inactive-value="0"
                    v-model="formValidate.is_host"
                    size="large"
                    active-text="开启"
                    inactive-text="关闭"
                  >
                  </el-switch>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="活动状态：" props="is_show" label-for="is_show">
                  <el-switch
                    class="defineSwitch"
                    :active-value="1"
                    :inactive-value="0"
                    v-model="formValidate.is_show"
                    size="large"
                    active-text="开启"
                    inactive-text="关闭"
                  >
                  </el-switch>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="规格选择：">
                  <el-table
                    ref="multipleTable"
                    :data="specsData"
                    :row-key="getRowKeys"
                    border
                    @selection-change="changeCheckbox"
                  >
                    <el-table-column type="selection" :reserve-selection="true" width="55"> </el-table-column>
                    <el-table-column
                      :label="item.title"
                      :min-width="item.minWidth"
                      v-for="(item, index) in columns"
                      :key="index"
                    >
                      <template slot-scope="scope">
                        <template v-if="item.key">
                          <div>
                            <span>{{ scope.row[item.key] }}</span>
                          </div>
                        </template>
                        <template v-else-if="item.slot === 'pic'">
                          <div
                            class="acea-row row-middle row-center-wrapper"
                            v-db-click
                            @click="modalPicTap('dan', 'danTable', scope.$index)"
                          >
                            <div class="pictrue pictrueTab" v-if="scope.row.pic">
                              <img v-lazy="scope.row.pic" />
                            </div>
                            <div class="upLoad pictrueTab acea-row row-center-wrapper" v-else>
                              <i class="el-icon-picture-outline" style="font-size: 24px"></i>
                            </div>
                          </div>
                        </template>
                        <template v-else-if="item.slot === 'price'">
                          <el-input-number
                            :controls="false"
                            v-model="scope.row.price"
                            :min="0"
                            :precision="2"
                            class="priceBox"
                            :active-change="false"
                          ></el-input-number>
                        </template>
                        <template v-if="item.slot === 'quota'">
                          <el-input-number
                            :controls="false"
                            v-model="scope.row.quota"
                            :min="1"
                            active-change
                            class="priceBox"
                          ></el-input-number>
                        </template>
                      </template>
                    </el-table-column>
                  </el-table>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row v-show="current === 2">
              <el-col :span="24">
                <el-form-item label="内容：">
                  <WangEditor
                    style="width: 90%"
                    :content="formValidate.description"
                    @editorContent="getEditorContent"
                  ></WangEditor>
                </el-form-item>
              </el-col>
            </el-row>
            <el-form-item>
              <el-button
                class="submission"
                v-db-click
                @click="step"
                :disabled="($route.params.id && current === 1) || current === 0"
                >上一步</el-button
              >
              <el-button
                type="primary"
                :disabled="submitOpen && current === 2"
                class="submission"
                v-db-click
                @click="next('formValidate')"
                >{{ current === 2 ? '提交' : '下一步' }}</el-button
              >
            </el-form-item>
          </el-form>
        </el-col>
      </el-row>
    </el-card>
    <!-- 选择商品-->
    <el-dialog :visible.sync="modals" title="商品列表" class="paymentFooter" width="1000px">
      <goods-list ref="goodslist" @getProductId="getProductId"></goods-list>
    </el-dialog>
    <!-- 上传图片-->
    <el-dialog :visible.sync="modalPic" width="950px" title="上传商品图" :close-on-click-modal="false">
      <uploadPictures
        :isChoice="isChoice"
        @getPic="getPic"
        @getPicD="getPicD"
        :gridBtn="gridBtn"
        :gridPic="gridPic"
        v-if="modalPic"
      ></uploadPictures>
    </el-dialog>
    <!-- 运费模板-->
    <freight-template ref="template" @addSuccess="productGetTemplate"></freight-template>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import goodsList from '@/components/goodsList/index';
import WangEditor from '@/components/wangEditor/index.vue';
import uploadPictures from '@/components/uploadPictures';
import { combinationInfoApi, combinationCreatApi, productAttrsApi } from '@/api/marketing';
import { productGetTemplateApi } from '@/api/product';
import freightTemplate from '@/components/freightTemplate/index';
import steps from '@/components/steps/index';

export default {
  name: 'storeCombinationCreate',
  components: {
    goodsList,
    uploadPictures,
    WangEditor,
    freightTemplate,
    steps,
  },
  data() {
    return {
      submitOpen: false,
      spinShow: false,
      isChoice: '',
      current: 0,
      modalPic: false,
      grid: {
        xl: 12,
        lg: 20,
        md: 24,
        sm: 24,
        xs: 24,
      },
      grid2: {
        xl: 8,
        lg: 8,
        md: 12,
        sm: 24,
        xs: 24,
      },
      gridPic: {
        xl: 6,
        lg: 8,
        md: 12,
        sm: 12,
        xs: 12,
      },
      gridBtn: {
        xl: 4,
        lg: 8,
        md: 8,
        sm: 8,
        xs: 8,
      },
      stepList: ['选择拼团商品', '填写基础信息', '修改商品详情'],
      myConfig: {
        autoHeightEnabled: false, // 编辑器不自动被内容撑高
        initialFrameHeight: 500, // 初始容器高度
        initialFrameWidth: '100%', // 初始容器宽度
        UEDITOR_HOME_URL: '/UEditor/',
        serverUrl: '',
      },
      modals: false,
      modal_loading: false,
      images: [],
      templateList: [],
      columns: [],
      specsData: [],
      picTit: '',
      tableIndex: 0,
      formValidate: {
        images: [],
        info: '',
        title: '',
        image: '',
        unit_name: '',
        price: 0,
        effective_time: 24,
        stock: 1,
        sales: 0,
        sort: 0,
        is_postage: 0,
        is_commission: 0,
        is_host: 0,
        is_show: 0,
        section_time: [],
        description: '',
        id: 0,
        product_id: 0,
        people: 2,
        once_num: 1,
        num: 1,
        temp_id: '',
        attrs: [],
        items: [],
        virtual: 100,
        virtualPeople: 0,
        head_commission: 0,
        logistics: ['1'], //选择物流方式
        freight: 2, //运费设置
        postage: 1, //设置运费金额
      },
      ruleValidate: {
        image: [{ required: true, message: '请选择主图', trigger: 'change' }],
        images: [
          {
            required: true,
            type: 'array',
            message: '请选择主图',
            trigger: 'change',
          },
          {
            type: 'array',
            min: 1,
            message: 'Choose two hobbies at best',
            trigger: 'change',
          },
        ],
        title: [{ required: true, message: '请输入拼团名称', trigger: 'blur' }],
        info: [{ required: true, message: '请输入拼团简介', trigger: 'blur' }],
        section_time: [
          {
            required: true,
            type: 'array',
            message: '请选择活动时间',
            trigger: 'change',
          },
        ],
        unit_name: [{ required: true, message: '请输入单位', trigger: 'blur' }],
        price: [
          {
            required: true,
            type: 'number',
            message: '请输入拼团价',
            trigger: 'blur',
          },
        ],
        cost: [
          {
            required: true,
            type: 'number',
            message: '请输入成本价',
            trigger: 'blur',
          },
        ],
        stock: [
          {
            required: true,
            type: 'number',
            message: '请输入库存',
            trigger: 'blur',
          },
        ],
        give_integral: [
          {
            required: true,
            type: 'number',
            message: '请输入赠送积分',
            trigger: 'blur',
          },
        ],
        effective_time: [
          {
            required: true,
            type: 'number',
            message: '请输入拼团时效(单位 小时)',
            trigger: 'blur',
          },
        ],
        people: [
          {
            required: true,
            type: 'number',
            message: '请输入拼团人数',
            trigger: 'blur',
          },
        ],
        num: [
          {
            required: true,
            type: 'number',
            message: '请输入购买数量限制',
            trigger: 'blur',
          },
        ],
        once_num: [
          {
            required: true,
            type: 'number',
            message: '请输入单次购买数量限制',
            trigger: 'blur',
          },
        ],
        virtualPeople: [
          {
            required: true,
            type: 'number',
            message: '请输入虚拟成团补齐人数',
            trigger: 'blur',
          },
        ],
        temp_id: [
          {
            required: true,
            message: '请选择运费模板',
            trigger: 'change',
            type: 'number',
          },
        ],
      },
      copy: 0,
      description: '',
    };
  },
  computed: {
    ...mapState('media', ['isMobile']),
    labelWidth() {
      return this.isMobile ? undefined : '155px';
    },
    labelPosition() {
      return this.isMobile ? 'top' : 'right';
    },
  },
  mounted() {
    if (this.$route.params.id) {
      this.copy = this.$route.params.copy;
      this.current = 1;
      this.getInfo();
    }
    this.productGetTemplate();
  },
  methods: {
    changePrice(e, index) {
      this.$set(this.specsData[index], 'price', e);
    },
    getEditorContent(data) {
      this.description = data;
    },
    // 添加运费模板
    freight() {
      this.$refs.template.id = 0;
      this.$refs.template.isTemplate = true;
    },
    // 拼团规格；
    productAttrs(row) {
      let that = this;
      productAttrsApi(row.id, 3)
        .then((res) => {
          let data = res.data.info;
          let selection = {
            type: 'selection',
            width: 60,
            align: 'center',
          };
          that.specsData = data.attrs;
          that.specsData.forEach(function (item, index) {
            that.$set(that.specsData[index], 'id', index);
          });
          that.formValidate.items = data.items;
          that.columns = data.header;
          // that.columns.unshift(selection);
        })
        .catch((res) => {
          that.$message.error(res.msg);
        });
    },
    // 多选
    changeCheckbox(selection) {
      this.formValidate.attrs = selection;
    },
    // 获取运费模板；
    productGetTemplate() {
      productGetTemplateApi().then((res) => {
        this.templateList = res.data;
      });
    },
    // 表单验证
    validate(prop, status, error) {
      if (status === false) {
        this.$message.error(error);
      }
    },
    // 商品id
    getProductId(row) {
      this.modal_loading = false;
      this.modals = false;
      setTimeout(() => {
        this.formValidate = {
          images: row.slider_image,
          info: row.store_info,
          title: row.store_name,
          image: row.image,
          unit_name: row.unit_name,
          price: 0, // 不取商品中的原价
          effective_time: 24,
          stock: row.stock,
          sales: row.sales,
          sort: row.sort,
          is_postage: row.is_postage,
          is_commission: 0,
          is_host: row.is_hot,
          is_show: 0,
          section_time: [],
          description: row.description, // 不取商品中的
          id: 0,
          people: 2,
          num: 1,
          once_num: 1,
          product_id: row.id,
          temp_id: row.temp_id,
          virtual: 100,
          virtualPeople: 0,
          logistics: row.logistics, //选择物流方式
          freight: row.freight, //运费设置
          postage: row.postage, //设置运费金额
          custom_form: row.custom_form, //自定义表单数据
          virtual_type: row.virtual_type, //虚拟商品类型
          head_commission: 0,
        };
        this.productAttrs(row);
      }, 500);
    },
    cancel() {
      this.modals = false;
    },
    // 具体日期
    onchangeTime(e) {
      this.formValidate.section_time = e;
    },
    // 详情
    getInfo() {
      this.spinShow = true;
      combinationInfoApi(this.$route.params.id)
        .then(async (res) => {
          let that = this;
          let info = res.data.info;
          let selection = {
            type: 'selection',
            width: 60,
            align: 'center',
          };
          this.formValidate = info;
          this.formValidate.virtualPeople = parseInt(
            this.formValidate.people - this.formValidate.people * (this.formValidate.virtual / 100),
          );
          this.$set(this.formValidate, 'items', info.attrs.items);
          this.columns = info.attrs.header;
          // this.columns.unshift(selection);
          this.specsData = info.attrs.value;
          that.specsData.forEach(function (item, index) {
            that.$set(that.specsData[index], 'id', index);
          });
          let data = info.attrs;
          let attr = [];
          for (let index in info.attrs.value) {
            if (info.attrs.value[index]._checked) {
              attr.push(info.attrs.value[index]);
            }
          }
          that.formValidate.attrs = attr;
          attr.forEach((row) => {
            that.$refs.multipleTable.toggleRowSelection(row, true);
          });
          this.spinShow = false;
        })
        .catch((res) => {
          this.spinShow = false;
          this.$message.error(res.msg);
        });
    },
    getRowKeys(row) {
      return row.id;
    },
    // 下一步
    next(name) {
      let that = this;
      if (this.current === 2) {
        this.formValidate.description = this.description;
        this.$refs[name].validate((valid) => {
          if (valid) {
            if (this.copy == 1) this.formValidate.copy = 1;
            this.formValidate.id = Number(this.$route.params.id) || 0;
            this.submitOpen = true;
            this.formValidate.virtual = parseInt(
              ((this.formValidate.people - this.formValidate.virtualPeople) / this.formValidate.people) * 100,
            );
            combinationCreatApi(this.formValidate)
              .then(async (res) => {
                this.submitOpen = false;
                this.$message.success(res.msg);
                setTimeout(() => {
                  this.$router.push({
                    path: this.$routeProStr + '/marketing/store_combination/index',
                  });
                }, 500);
              })
              .catch((res) => {
                this.submitOpen = false;
                this.$message.error(res.msg);
              });
          } else {
            return false;
          }
        });
      } else if (this.current === 1) {
        this.$refs[name].validate((valid) => {
          if (valid) {
            if (that.formValidate.people < 2) {
              return that.$message.error('拼团人数必须大于2');
            }
            if (that.formValidate.num < 0) {
              return that.$message.error('购买数量限制必须大于0');
            }
            if (that.formValidate.once_num < 0) {
              return that.$message.error('单次购买数量限制必须大于0');
            }
            if (!that.formValidate.attrs) {
              return that.$message.error('请选择属性规格');
            } else {
              for (let index in that.formValidate.attrs) {
                if (that.formValidate.attrs[index].quota <= 0) {
                  return that.$message.error('拼团限量必须大于0');
                }
                if (this.formValidate.attrs[index].quota > this.formValidate.attrs[index]['stock']) {
                  return this.$message.error('拼团限量不能超过规格库存');
                }
              }
            }
            this.current += 1;
          } else {
            return this.$message.warning('请完善您的信息');
          }
        });
      } else {
        if (this.formValidate.image) {
          this.current += 1;
        } else {
          this.$message.warning('请选择商品');
        }
      }
    },
    // 上一步
    step() {
      this.current--;
    },
    // 内容
    getContent(val) {
      this.formValidate.description = val;
    },
    // 点击商品图
    modalPicTap(tit, picTit, index) {
      this.modalPic = true;
      this.isChoice = tit === 'dan' ? '单选' : '多选';
      this.picTit = picTit;
      this.tableIndex = index;
    },
    // 获取单张图片信息
    getPic(pc) {
      switch (this.picTit) {
        case 'danFrom':
          this.formValidate.image = pc.att_dir;
          break;
        default:
          if (!!this.formValidate.attrs && this.formValidate.attrs.length) {
            this.$set(this.specsData[this.tableIndex], '_checked', true);
          }
          this.specsData[this.tableIndex].pic = pc.att_dir;
      }
      this.modalPic = false;
    },
    // 获取多张图信息
    getPicD(pc) {
      this.images = pc;
      this.images.map((item) => {
        this.formValidate.images.push(item.att_dir);
        this.formValidate.images = this.formValidate.images.splice(0, 10);
      });
      this.modalPic = false;
    },
    handleRemove(i) {
      this.images.splice(i, 1);
      this.formValidate.images.splice(i, 1);
    },
    // 选择商品
    changeGoods() {
      this.modals = true;
      this.$nextTick((e) => {
        this.$refs.goodslist.formValidate.is_show = -1;
        this.$refs.goodslist.formValidate.type = 3;
        this.$refs.goodslist.getList();
        this.$refs.goodslist.goodsCategory();
      });
    },
    // 移动
    handleDragStart(e, item) {
      this.dragging = item;
    },
    handleDragEnd(e, item) {
      this.dragging = null;
    },
    // 首先把div变成可以放置的元素，即重写dragenter/dragover
    handleDragOver(e) {
      e.dataTransfer.dropEffect = 'move';
    },
    handleDragEnter(e, item) {
      e.dataTransfer.effectAllowed = 'move';
      if (item === this.dragging) {
        return;
      }
      const newItems = [...this.formValidate.images];
      const src = newItems.indexOf(this.dragging);
      const dst = newItems.indexOf(item);
      newItems.splice(dst, 0, ...newItems.splice(src, 1));
      this.formValidate.images = newItems;
    },
  },
};
</script>

<style lang="scss" scoped>
.content_width {
  width: 460px;
}
.grey {
  font-size: 12px;
  color: #999;
}
.maxW ::v-deep .ivu-select-dropdown {
  max-width: 600px;
}
.ivu-table-wrapper {
  border-left: 1px solid #dcdee2;
  border-top: 1px solid #dcdee2;
}
.tabBox_img {
  width: 50px;
  height: 50px;
}
.tabBox_img img {
  width: 100%;
  height: 100%;
}
.priceBox {
  width: 100%;
}
.form {
  .picBox {
    display: inline-block;
    cursor: pointer;
  }
  .pictrue {
    width: 60px;
    height: 60px;
    border: 1px dotted rgba(0, 0, 0, 0.1);
    margin-right: 15px;
    display: inline-block;
    position: relative;
    cursor: pointer;

    img {
      width: 100%;
      height: 100%;
    }
    .btndel {
      position: absolute;
      z-index: 9;
      width: 20px !important;
      height: 20px !important;
      left: 46px;
      top: -4px;
    }
  }
  .upLoad {
    width: 58px;
    height: 58px;
    line-height: 58px;
    border: 1px dotted rgba(0, 0, 0, 0.1);
    border-radius: 4px;
    background: rgba(0, 0, 0, 0.02);
    cursor: pointer;
  }
  .col {
    color: #2d8cf0;
    cursor: pointer;
  }
}
.addfont {
  font-size: 12px;
  color: var(--prev-color-primary);
  margin-left: 14px;
  cursor: pointer;
  margin-left: 10px;
  cursor: pointer;
}
</style>
