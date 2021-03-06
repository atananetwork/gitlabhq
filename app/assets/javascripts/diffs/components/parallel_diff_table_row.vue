<script>
import $ from 'jquery';
import { mapGetters } from 'vuex';
import DiffTableCell from './diff_table_cell.vue';
import {
  NEW_LINE_TYPE,
  OLD_LINE_TYPE,
  CONTEXT_LINE_TYPE,
  CONTEXT_LINE_CLASS_NAME,
  OLD_NO_NEW_LINE_TYPE,
  PARALLEL_DIFF_VIEW_TYPE,
  NEW_NO_NEW_LINE_TYPE,
  LINE_POSITION_LEFT,
  LINE_POSITION_RIGHT,
} from '../constants';

export default {
  components: {
    DiffTableCell,
  },
  props: {
    fileHash: {
      type: String,
      required: true,
    },
    contextLinesPath: {
      type: String,
      required: true,
    },
    line: {
      type: Object,
      required: true,
    },
    isBottom: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      isLeftHover: false,
      isRightHover: false,
    };
  },
  computed: {
    ...mapGetters('diffs', ['isParallelView']),
    isContextLine() {
      return this.line.left.type === CONTEXT_LINE_TYPE;
    },
    classNameMap() {
      return {
        [CONTEXT_LINE_CLASS_NAME]: this.isContextLine,
        [PARALLEL_DIFF_VIEW_TYPE]: this.isParallelView,
      };
    },
    parallelViewLeftLineType() {
      if (this.line.right.type === NEW_NO_NEW_LINE_TYPE) {
        return OLD_NO_NEW_LINE_TYPE;
      }

      return this.line.left.type;
    },
  },
  created() {
    this.newLineType = NEW_LINE_TYPE;
    this.oldLineType = OLD_LINE_TYPE;
    this.linePositionLeft = LINE_POSITION_LEFT;
    this.linePositionRight = LINE_POSITION_RIGHT;
    this.parallelDiffViewType = PARALLEL_DIFF_VIEW_TYPE;
  },
  methods: {
    handleMouseMove(e) {
      const isHover = e.type === 'mouseover';
      const hoveringCell = e.target.closest('td');
      const allCellsInHoveringRow = Array.from(e.currentTarget.children);
      const hoverIndex = allCellsInHoveringRow.indexOf(hoveringCell);

      if (hoverIndex >= 2) {
        this.isRightHover = isHover;
      } else {
        this.isLeftHover = isHover;
      }
    },
    // Prevent text selecting on both sides of parallel diff view
    // Backport of the same code from legacy diff notes.
    handleParallelLineMouseDown(e) {
      const line = $(e.currentTarget);
      const table = line.closest('table');

      table.removeClass('left-side-selected right-side-selected');
      const [lineClass] = ['left-side', 'right-side'].filter(name => line.hasClass(name));

      if (lineClass) {
        table.addClass(`${lineClass}-selected`);
      }
    },
  },
};
</script>

<template>
  <tr
    :class="classNameMap"
    class="line_holder"
    @mouseover="handleMouseMove"
    @mouseout="handleMouseMove"
  >
    <diff-table-cell
      :file-hash="fileHash"
      :context-lines-path="contextLinesPath"
      :line="line"
      :line-type="oldLineType"
      :line-position="linePositionLeft"
      :is-bottom="isBottom"
      :is-hover="isLeftHover"
      :show-comment-button="true"
      :diff-view-type="parallelDiffViewType"
      class="diff-line-num old_line"
    />
    <td
      v-once
      :id="line.left.lineCode"
      :class="parallelViewLeftLineType"
      class="line_content parallel left-side"
      @mousedown.native="handleParallelLineMouseDown"
      v-html="line.left.richText"
    >
    </td>
    <diff-table-cell
      :file-hash="fileHash"
      :context-lines-path="contextLinesPath"
      :line="line"
      :line-type="newLineType"
      :line-position="linePositionRight"
      :is-bottom="isBottom"
      :is-hover="isRightHover"
      :show-comment-button="true"
      :diff-view-type="parallelDiffViewType"
      class="diff-line-num new_line"
    />
    <td
      v-once
      :id="line.right.lineCode"
      :class="line.right.type"
      class="line_content parallel right-side"
      @mousedown.native="handleParallelLineMouseDown"
      v-html="line.right.richText"
    >
    </td>
  </tr>
</template>
