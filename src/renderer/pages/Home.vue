<template>
  <div id="home">
    <div class="menu">
      <router-link to="/settings">
        <span class="icon">
          <i class="fa fa-cog"></i>
        </span>
        {{ 'menu.settings'|trans }}
      </router-link>
    </div>
    <!--
      Unity departments
    -->
    <article v-if="page=='departments'">
      <header>
        <h1>
          Departamentos
        </h1>
      </header>
      <section>
        <p>Escolha o departamento aonde deseja atendimento</p>
        <div class="columns is-multiline is-mobile">
          <div class="column is-6" v-for="department in departments" :key="department.id">
            <button type="button" class="button is-xlarge is-block" @click="selectDepartment(department)">
              {{department.nome}}
            </button>
          </div>
        </div>
      </section>
      <footer>
      </footer>
    </article>

    <!--
      Department services
    -->
    <article v-if="page=='department'">
      <header>
        <h1>{{department.nome}}</h1>
      </header>
      <section>
        <p>Escolha o serviço desejado</p>
        <div class="columns is-multiline is-mobile">
          <div class="column is-6" v-for="service in services" :key="service.id">
            <button type="button" class="button is-xlarge is-block" @click="selectService(service)">
              {{service.nome}}
            </button>
          </div>
        </div>
      </section>
      <footer>
        <button type="button" class="button is-large" @click="begin">
          <span class="icon is-small">
            <i class="fa fa-chevron-left"></i>
          </span>
          <span>Voltar</span>
        </button>

        <span class="tag is-info is-large is-pulled-right	" v-if="timer">
          {{timer}}s
        </span>
      </footer>
    </article>

    <!--
      All services
    -->
    <article v-if="page=='allServices'">
      <header>
        <h1>Serviços</h1>
      </header>
      <section>
        <p>Escolha o serviço desejado</p>
        <div class="columns is-multiline is-mobile">
          <div class="column is-6" v-for="su in enabledServices" :key="su.servico.id">
            <button type="button" class="button is-primary is-xlarge is-block" @click="selectService(su)">
              {{su.servico.nome}}
            </button>
          </div>
        </div>
      </section>
      <footer>
      </footer>
    </article>

    <!--
      Service overview
    -->
    <article v-if="page=='service'">
      <header>
        <h1>{{servicoUnidade.servico.nome}}</h1>
      </header>
      <section>
        <div class="subservices" v-if="subservices.length">
          <p>Atividades executadas pelo serviço escolhido</p>
          <ul class="columns is-multiline is-mobile">
            <li v-for="subservice in subservices" :key="subservice.id" class="column is-6">
              {{subservice.nome}}
            </li>
          </ul>
        </div>
        <div class="columns is-multiline is-mobile">
          <div class="column is-6">
            <button type="button" class="button is-info is-xlarge is-block" @click="ticket(null)">
              Normal
            </button>
          </div>
          <div class="column is-6">
            <button type="button" class="button is-danger is-xlarge is-block" @click="selectPriority">
              Prioridade
            </button>
          </div>
        </div>
      </section>
      <footer>
        <button type="button" class="button is-large" @click="begin">
          <span class="icon is-small">
            <i class="fa fa-chevron-left"></i>
          </span>
          <span>Voltar</span>
        </button>

        <span class="tag is-info is-large is-pulled-right	" v-if="timer">
          {{timer}}s
        </span>
      </footer>
    </article>

    <!--
      Priorities
    -->
    <article v-if="page=='priorities'">
      <header>
        <h1>{{servicoUnidade.servico.nome}}</h1>
      </header>
      <section>
        <div class="columns is-multiline is-mobile">
          <div class="column is-6" v-for="priority in priorities" :key="priority.id">
            <button type="button" class="button is-danger is-xlarge is-block" @click="ticket(priority)">
              {{priority.nome}}
            </button>
          </div>
        </div>
      </section>
      <footer>
        <button type="button" class="button is-large" @click="begin">
          <span class="icon is-small">
            <i class="fa fa-chevron-left"></i>
          </span>
          <span>Voltar</span>
        </button>

        <span class="tag is-info is-large is-pulled-right	" v-if="timer">
          {{timer}}s
        </span>
      </footer>
    </article>

    <!--
      Printing
    -->
    <article v-if="page=='printing'" class="printing">
      <header>
        <h1>
          Imprimindo
        </h1>
      </header>
      <section>
        <p>Por favor aguarde a impressão da sua senha de atendimento</p>
        <div class="columns is-mobile is-centered" v-if="ticketInfo">
          <div class="column is-half is-narrow">
            <div class="tile is-child notification is-warning is-vertical">
              <p class="subtitle">{{ticketInfo.prioridade.nome}}</p>
              <p class="title">{{ticketInfo.senha.format}}</p>
              <p class="subtitle">{{ticketInfo.servico.nome}}</p>
            </div>
          </div>
        </div>
      </section>
      <footer>
        <button type="button" class="button is-large" @click="begin">
          <span class="icon is-small">
            <i class="fa fa-chevron-left"></i>
          </span>
          <span>Voltar</span>
        </button>

        <span class="tag is-info is-large is-pulled-right	" v-if="timer">
          {{timer}}s
        </span>
      </footer>
    </article>

    <iframe id="frame-impressao" width="0" height="0" style="border:none;"></iframe>
  </div>
</template>

<script>
  import auth from '@/store/modules/auth'

  let running = false
  let intervalId = 0
  let timeoutId = 0

  function isExpired ($store) {
    return auth.getters.isExpired($store.state.auth)
  }

  function checkToken ($store) {
    clearTimeout(timeoutId)

    if (!running) {
      console.log('not running')
      return
    }

    console.log('checking token. Authenticated: ' + $store.getters.isAuthenticated)

    if ($store.getters.isAuthenticated && isExpired($store)) {
      console.log('token expired, refreshing')
      $store
        .dispatch('refresh')
        .then(() => {
          console.log('token refreshed')
        }, e => {
          console.log(e)
        })
    }

    timeoutId = setTimeout(() => {
      checkToken($store)
    }, 60 * 1000)
}

  export default {
    name: 'home',
    data () {
      return {
        firstPage: 'allServices',
        page: '',
        enabledServices: [],
        servicoUnidade: null,
        department: null,
        departments: [],
        services: [],
        subservices: [],
        priorities: [],
        customer: {
          id: '',
          name: ''
        },
        ticketInfo: null,
        timer: null,
        startTime: 15
      }
    },
    methods: {
      begin () {
        this.page = this.firstPage
        this.department = null
        this.service = null
        this.ticketInfo = null
        this.timer = null
      },

      tick () {
        if (this.page !== this.firstPage) {
          if (this.timer === null) {
            this.timer = this.startTime
            console.log('[timer] started')
          } else {
            this.timer--
            console.log('[timer] tick')
          }
          if (this.timer <= 0) {
            this.timer = null
            this.page = this.firstPage
            console.log('[timer] restart!')
          }
        }
      },

      selectDepartment (department) {
        this.page = 'department'
        this.department = department
      },

      selectService (su) {
        this.page = 'service'
        this.servicoUnidade = su
      },

      selectPriority () {
        if (this.priorities.length > 1) {
          this.page = 'priorities'
        } else {
          this.ticket(this.priorities[0])
        }
      },

      ticket (priority) {
        const data = {
          unityId: this.$store.state.config.unity,
          serviceId: this.servicoUnidade.servico.id,
          priorityId: (priority ? priority.id : 1),
          customer: this.customer
        }
        this.$store.dispatch('newTicket', data).then((ticket) => {
          this.print(ticket)
        })
      },

      print (ticket) {
        this.page = 'printing'
        this.ticketInfo = ticket
        this.$store.dispatch('print', ticket).then((response) => {
          const iframe = document.getElementById('frame-impressao')
          iframe.src = 'data:text/html;charset=utf-8,' + response
        })
      }
    },
    beforeMount () {
      const store = this.$store
      const config = store.state.config

      if (!config || !config.server) {
        this.$router.push('/settings')
        return
      }
      if (!running) {
        running = true
        checkToken(store)
      }

      store.dispatch('fetchServices', config.unity).then((services) => {
        this.enabledServices = []
        for (var j = 0; j < config.services.length; j++) {
          let id = config.services[j]
          for (var i = 0; i < services.length; i++) {
            let su = services[i]
            if (su.servico.id === id) {
              this.enabledServices.push(su)
              break
            }
          }
        }
        if (this.enabledServices.length > 1) {
          this.firstPage = 'allServices'
        } else {
          this.firstPage = 'service'
          this.servicoUnidade = this.enabledServices[0]
        }

        this.begin()
      })

      store.dispatch('fetchPriorities', config.unity).then((priorities) => {
        this.priorities = priorities.filter((p) => {
          return p.peso > 0
        })
      })

      this.intervalId = setInterval(() => {
        this.tick()
      }, 1000)
    },
    beforeDestroy () {
      running = false
      clearInterval(intervalId)
      clearTimeout(timeoutId)
    },
    watch: {
      page () {
        this.timer = null
      }
    }
  }
</script>

<style lang="sass">
  #home .menu
    position: fixed
    top: 2vh
    left: 2vw
    background-color: rgba(0,0,0,.5)
    padding: 10px
    border-radius: 8px
    opacity: 0
    z-index: 100
    &:hover
      opacity: 1
      transition: opacity 0.2s ease-in-out
      background-color: rgba(0,0,0,1)
    a
      color: #ffffff

  article
    width: 100%
    height: 100%
    top: 0
    left: 0
    position: fixed
    header
      padding: 1.5rem 2rem
      background-color: #4fc08d
      h1
        color: #ffffff
        font-size: 3rem
        font-weight: bold
        padding: .5rem 0
        text-align: center
        background-image: url('/static/images/logo.png')
        background-repeat: no-repeat
        background-size: contain
    section
      padding: 2rem
      p
        text-align: center
        font-size: 1.5rem
    &.printing
      .is-centered
        padding-top: 2rem
        .tile
          .title
            font-size: 5rem
          .subtitle
            font-size: 2.5rem
    footer
      width: 100%
      padding: 2rem
      position: fixed
      bottom: 0
      background-color: #f1f1f1
    .is-block
      width: 100%
      margin-top: 1rem
    .is-xlarge
      font-size: 2.25rem
      line-height: 1
      border-radius: .3rem

    .subservices
      font-size: 1.5rem
      ul
        padding: 2rem 0
</style>