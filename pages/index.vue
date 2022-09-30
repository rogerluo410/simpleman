<template>
  <div class="home">
    <div class="trytopnav">
      <div class="w3-bar" style="overflow: auto">
        <a-row type="flex" align="middle">
          <div class="icon-panel">
            <svg-icon name="save" class="icon-size" />
          </div>
          <div class="icon-panel">
            <svg-icon name="rotate" class="icon-size" />
          </div>
          <div class="icon-panel">
            <svg-icon name="adjust" class="icon-size" />
          </div>
        </a-row>

        <a-button
          :disabled="!enableTryit()"
          class="run-btn"
          :loading="loading"
          @click="submitRequest"
        >
          运行 ❯
        </a-button>
      </div>
    </div>

    <div id="container">
      <div id="textareacontainer">
        <div id="textarea">
          <div id="textareawrapper">
            <div class="request-panel">
              <div class="request-method">{{ request_method }}</div>
              <div class="request-url">
                <p>{{ request_url }}</p>
              </div>
            </div>
            <!-- if Nust.js/SSR（如果在 Nuxt.js 环境下，需要外面包裹一层 no-ssr） -->
            <no-ssr placeholder="Codemirror Loading...">
              <div class="code-mirror-box">
                <LazyJsonEditor
                  ref="jsonEditor"
                  :json-code="jsonCode"
                  :readonly="readonly"
                  @change="handleJsonChange"
                />
              </div>
            </no-ssr>
          </div>
        </div>
      </div>
      <div id="iframecontainer">
        <div id="iframe">
          <div id="iframewrapper">
            <div class="request-panel">
              <div class="request-url">
                <p>
                  结果:
                  <span v-show="response_status"
                    >状态码 {{ response_status }}</span
                  >
                </p>
              </div>
            </div>
            <!-- if Nust.js/SSR（如果在 Nuxt.js 环境下，需要外面包裹一层 no-ssr） -->
            <no-ssr placeholder="Codemirror Loading...">
              <div class="code-mirror-box">
                <LazyJsonEditor
                  ref="jsonResultEditor"
                  :json-code="jsonResultCode"
                  :readonly="true"
                />
              </div>
            </no-ssr>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Axios from "axios";

export default {
  layout: "tryit",
  data() {
    return {
      loading: false,
      jsonCode: "{}",
      jsonResultCode: "{}",
      readonly: false,
      token: "",
      request_url: "",
      request_method: "",
      response_status: "",
    };
  },
  mounted() {
    // 刷新jsonEditor
    this.$nextTick(() => {
      this.$refs.jsonEditor && this.$refs.jsonEditor.refresh();
      this.$refs.jsonResultEditor && this.$refs.jsonResultEditor.refresh();
    });
  },
  methods: {
    enableTryit() {
      return true;
    },
    handleJsonChange(val) {
      if (this.jsonCode !== val) {
        this.jsonCode = val;
      }
    },
    async submitRequest() {
      this.loading = true;
      const method = this.request_method.toLowerCase();
      try {
        const res = await Axios[method](this.request_url, this.jsonCode, {
          headers: {
            token: this.token,
          },
        });

        this.loading = false;
        this.jsonResultCode = JSON.stringify(res.data);
        this.response_status = res.status;
      } catch (error) {
        if (error.response) {
          // Get the error status, inform the user of the error.
          this.jsonResultCode = JSON.stringify(error.response.data);
          this.response_status = error.response.status;
        }

        this.loading = false;

        // Unexpected error, tell the user to try again later.
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.trytopnav {
  height: 48px;
  overflow: hidden;
  min-width: 380px;
  position: absolute;
  width: 100%;
  background-color: #e7e9eb;
}

.w3-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 50%;
  height: 100%;
}

#container {
  background-color: #e7e9eb;
  width: 100%;
  overflow: auto;
  position: absolute;
  top: 48px;
  bottom: 0;
  height: auto;
}

#textareacontainer,
#iframecontainer {
  float: left;
  height: 100%;
  width: 50%;
}

#textarea {
  padding-left: 10px;
}

#textareawrapper {
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: #ffffff;
  position: relative;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 12%), 0 1px 2px rgba(0, 0, 0, 24%);
}

.code-mirror-box {
  overflow-y: scroll;
  overflow-x: auto;
}

#textareaCode,
#textareaCode2 {
  background-color: #ffffff;
  font-family: consolas, Menlo, "courier new", monospace;
  font-size: 15px;
  height: 100%;
  width: 100%;
  padding: 8px;
  resize: none;
  border: none;
  line-height: normal;
}

textarea {
  overflow: auto;
}

#textareacontainer,
#iframecontainer {
  float: left;
  height: 100%;
  width: 50%;
}

#iframe {
  padding-left: 5px;
  padding-right: 10px;
  position: relative;
}

#textarea,
#textarea2,
#iframe {
  height: 100%;
  width: 100%;
  padding-bottom: 10px;
  padding-top: 1px;
}

#iframewrapper {
  width: 100%;
  height: 100%;
  -webkit-overflow-scrolling: touch;
  background-color: #ffffff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 12%), 0 1px 2px rgba(0, 0, 0, 24%);
}

#iframeResult,
#iframeSource {
  background-color: #ffffff;
  height: 100%;
  width: 100%;
}

.run-btn {
  font-size: 16px;
  text-transform: uppercase;
  letter-spacing: 1px;
  fill: #ffffff;
  color: #ffffff;
  background-color: #0277bd;
  border-radius: 10px;
  box-shadow: 0px 21px 11px -10px rgba(0, 0, 0, 0.2);
}

button:disabled,
button[disabled] {
  border: 1px solid #999999;
  background-color: #cccccc;
  color: #666666;
}

.request-panel {
  height: 30px;
  border-bottom: 1px #acacac solid;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-flow: row wrap;
}

.request-method {
  display: flex;
  align-items: center;
  width: 55px;
  height: 100%;
  padding: 0 6px;
  border-right: 1px #acacac solid;
}

.request-url {
  padding: 0 6px;
}
</style>
