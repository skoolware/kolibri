<script>

  import get from 'lodash/get';

  export default {
    name: 'CoreTable',
    props: {
      selectable: {
        type: Boolean,
        default: false,
        required: false,
      },
      emptyMessage: {
        type: String,
        default: null,
      },
    },
    computed: {
      tHeadStyle() {
        return {
          borderBottom: `solid 1px ${this.$themeTokens.fineLine}`,
          fontSize: '12px',
          color: this.$themeTokens.annotation,
        };
      },
      tbodyTrStyle() {
        const selectable = {
          cursor: 'pointer',
          ':hover': {
            backgroundColor: this.$themePalette.grey.v_100,
          },
        };
        return Object.assign(
          {
            ':not(:last-child)': {
              borderBottom: `solid 1px ${this.$themeTokens.fineLine}`,
            },
          },
          this.selectable ? selectable : {}
        );
      },
    },
    render(createElement) {
      let tableHasRows = true;

      // create <thead> element with #headers slot
      const theadEl = createElement('thead', { style: this.tHeadStyle }, [
        createElement('tr', {}, this.$slots.headers),
      ]);

      const tbodyCopy = [...this.$slots.tbody];
      tbodyCopy.forEach(tbody => {
        // Need to check componentOptions if wrapped in <transition-group>, or just children
        // if in regular <tbody>
        const tgroupChildren = get(tbody, 'componentOptions.children');
        if (tgroupChildren) {
          tableHasRows = tgroupChildren.length > 0;
        }

        if (tbody.children) {
          tableHasRows = tbody.children.length > 0;
          tbody.children.forEach(child => {
            if (!child.data) {
              child.data = {};
            }
            if (!child.data.class) {
              child.data.class = [];
            } else if (child.data.class && !Array.isArray(child.data.class)) {
              child.data.class = [child.data.class];
            }
            child.data.class.push(this.$computedClass(this.tbodyTrStyle));
          });
        }
      });

      // Insert an empty message as a <p> at the end if it is provided and the
      // table has no rows.
      const showEmptyMessage = this.emptyMessage && !tableHasRows;

      return createElement('div', { class: 'core-table-container' }, [
        createElement('table', { class: 'core-table' }, [
          ...(this.$slots.default || []),
          theadEl,
          tbodyCopy,
        ]),
        showEmptyMessage && createElement('p', this.emptyMessage),
      ]);
    },
  };

</script>


<style lang="scss" scoped>

  @import '~kolibri-design-system/lib/styles/definitions';

  .core-table-container {
    @extend %momentum-scroll;

    overflow-x: auto;
    overflow-y: hidden;
  }

  .core-table {
    width: 100%;
    font-size: 14px;
  }

  /deep/ thead th {
    text-align: left;
    vertical-align: bottom;
  }

  /deep/ tr {
    text-align: left;
  }

  /deep/ th,
  /deep/ td {
    padding: 12px 8px;
    line-height: 1.5em;
    vertical-align: top;
  }

  /deep/ td {
    max-width: 300px;
    overflow-x: auto;
  }

  /deep/ .core-table-checkbox-col {
    width: 40px;
    .k-checkbox-container {
      margin: 0 0 0 2px;
      line-height: 1em;
    }
  }

  /deep/ .core-table-button-col {
    padding: 4px;
    text-align: right;

    button {
      margin: 0;
    }
  }

</style>
