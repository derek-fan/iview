<template>
  <Modal
    width="496"
    v-model.trim="cForm.modal"
    :title="cForm.id ? '编辑' : '新增'"
    :loading="cForm.loading"
    @on-ok="submit"
  >
    <Form
      class="c-form c-form--modal"
      ref="form"
      :model="cForm.model"
      :rules="cForm.rules"
      :label-width="80"
    >
      <Form-item label="标题" prop="title">
        <Input
          class="c-form__input"
          v-model.trim="cForm.model.title"
          placeholder="请输入标题"
        />
      </Form-item>
      <Form-item label="链接" prop="link">
        <Input
          class="c-form__input"
          v-model.trim="cForm.model.link"
          placeholder="请输入链接"
        />
      </Form-item>
      <Form-item label="图片" prop="pictureId">
        <c-uploader
          class="c-form__input"
          :key="`picture-${cForm.id}`"
          :default-file-ids="cForm.model.pictureId"
          @change="
            value => {
              handleFormUploaderChange('pictureId', value);
            }
          "
        />
      </Form-item>
      <Form-item label="状态" prop="status">
        <Select
          class="c-form__input"
          placeholder="请选择状态"
          clearable
          v-model="cForm.model.status"
        >
          <Option
            v-for="item in dicts.PublishStatus"
            :key="item.value"
            :value="item.value"
            :label="item.label"
          >
            {{ item.label }}
          </Option>
        </Select>
      </Form-item>
    </Form>
  </Modal>
</template>

<script>
import { Component, Vue } from "vue-property-decorator";
import FormMixin from "view-ui-admin/src/mixins/form";

const module = "ads";

@Component({
  mixins: [FormMixin]
})
export default class ListForm extends Vue {
  data() {
    return {
      cForm: {
        id: 0,
        modal: false,
        loading: true,
        model: this.getFormInitModel(),
        rules: {
          title: [
            {
              required: true,
              message: "标题不能为空"
            }
          ],
          pictureId: [
            {
              required: true,
              message: "图片不能为空"
            }
          ]
        }
      }
    };
  }

  getFormInitModel() {
    return {
      status: 1
    };
  }

  show(detail) {
    this.cForm.modal = true;

    if (detail && detail.id) {
      this.cForm.id = detail.id;
      this.initFormFields(detail);
    } else {
      this.cForm.id = 0;
    }
  }

  submit() {
    this.$refs.form.validate(async valid => {
      if (valid) {
        const { id, model } = this.cForm;
        const { data } = await this.$store.dispatch(
          `${module}/${id ? "put" : "post"}`,
          {
            id,
            body: model
          }
        );

        this.cForm.modal = false;
        this.$Message.success(`${id ? "编辑" : "新增"}成功`);
        this.$emit("get-list", data);
      }

      this.fixFormButtonLoading();
    });
  }
}
</script>
