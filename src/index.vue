<template>
  <div ref="container" class="vue-bpmn-diagram-container"></div>
</template>

<script>
  import BpmnJS from 'bpmn-js/dist/bpmn-navigated-viewer.production.min.js';

  export default {
    name: 'vue-bpmn-84',
    props: {
      url: {
        type: String,
        required: true
      },
      options: {
        type: Object
      }
    },
    data: function() {
      return {
        diagramXML: null
      };
    },
    mounted: function () {
      var container = this.$refs.container;

      var self = this;
      var _options = Object.assign({
        container: container
      }, this.options)
      this.bpmnViewer = new BpmnJS(_options);

      this.bpmnViewer.on('import.done', function(event) {

        var error = event.error;
        var warnings = event.warnings;

        if (error) {
          self.$emit('error', error);
        } else {
          self.$emit('shown', warnings);
        }

        self.bpmnViewer.get('canvas').zoom('fit-viewport');
      });

      if (this.url) {
        this.fetchDiagram(this.url);
      }
    },
    beforeDestroy: function() {
      this.bpmnViewer.destroy();
    },
    watch: {
      url: function(val) {
        this.$emit('loading');
        this.fetchDiagram(val);
      },
      diagramXML: function(val) {
        this.bpmnViewer.importXML(val);
      }
    },
    methods: {
      fetchDiagram: function(url) {

        var self = this;

        fetch(url)
          .then(function(response) {
            return response.text();
          })
          .then(function(text) {
            self.diagramXML = text;
          })
          .catch(function(err) {
            self.$emit('error', err);
          });
      }
    }
  };
</script>

<style>
  .vue-bpmn-diagram-container {
    height: 100%;
    width: 100%;
  }
</style>
