<template>
  <div class="frame" ref="container">
    <header class="tabs">
      <nav>
        <a class="active"><b-icon size="is-small" icon="information-outline" />Console</a>
        <a><b-icon size="is-small" icon="information-outline" />Terminal 1</a>

        <a class="tool"><b-icon size="is-small" icon="plus"/></a>
      </nav>
    </header>

    <div class="down">
      <div ref="term" class="term"></div>
    </div>
  </div>
</template>

<script lang="ts">
import 'xterm/css/xterm.css'
import { FitAddon } from 'xterm-addon-fit'
import { Terminal } from 'xterm'
import { Component, Vue } from 'vue-property-decorator'
import colors, { StyleFunction } from 'ansi-colors'

@Component
export default class Workspace extends Vue {
  public term = new Terminal({
    fontFamily: '"Fira Code", monospace',
    fontSize: 12,
    theme: {
      background: '#1e1e1e'
    }
  })

  fitAddon = new FitAddon()
  activeTerm = 0

  mounted() {
    const { term, fitAddon } = this
    term.loadAddon(fitAddon)
    console.log(this.$refs.term)
    term.open(this.$refs.term as HTMLDivElement)
    fitAddon.fit()
    setTimeout(() => fitAddon.fit(), 1000)
    term.writeln(colors.green('Welcome to Grapefruit!'))

    this.$ws.on('console', (level: string, msg: string) => {
      this.log(level, msg)
    })
  }

  log(level: string, ...args: string[]) {
    const { term } = this
    if (!term) return

    const color: { [key: string]: StyleFunction } = {
      info: colors.greenBright,
      error: colors.redBright,
      warn: colors.yellow,
      warning: colors.yellow
    }

    const renderer = color[level] || colors.whiteBright
    const ts = `[${new Date().toLocaleString()}]`
    const text = renderer([ts, ...args].join(' ').replace(/\n/g, '\r\n'))
    term.writeln(text)
  }

  resize() {
    this.fitAddon.fit()
  }

  beforeDestroy() {
    this.term.dispose()
  }

  tabChanged() {
    //
  }
}
</script>

<style lang="scss" scoped>
.frame {
  height: 100%;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.tabs {
  a {
    display: inline-block;
    color: #888;
    border-bottom: 4px solid transparent;

    &.tool {
      padding: 0.5em 0;
    }

    &:hover {
      color: #eee;
      border-bottom-color: #aaa;
    }

    &.active {
      border-bottom-color: #3a3a3a;
      &:hover {
        border-bottom-color: #aaa;
      }
    }
  }
}

.down {
  flex: 1;
  overflow: hidden;

  .term {
    height: 100%;
  }
}

.xterm-viewport {
  overflow-y: hidden;
}
</style>
