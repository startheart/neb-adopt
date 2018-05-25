<template>
  <div class="pet">
    <div class="container">
      <div class="row">
        <div class="col-xs-12 col-sm-8 col-sm-push-2">
          <h1 class="text-center">Pete's Pet Shop</h1>
          <hr/>
          <br/>
        </div>
      </div>

      <div id="petsRow" class="row">
        <!-- PETS LOAD HERE -->
      </div>
    </div>

    <div id="petTemplate" style="display: none">
      <div class="col-sm-6 col-md-4 col-lg-3">
        <div class="panel panel-default panel-pet">
          <div class="panel-heading">
            <h3 class="panel-title">Scrappy</h3>
          </div>
          <div class="panel-body">
            <img alt="140x140" data-src="holder.js/140x140" class="img-rounded img-center" style="width: 100%" src="https://animalso.com/wp-content/uploads/2017/01/Golden-Retriever_6.jpg" data-holder-rendered="true">
            <br/><br/>
            <strong>Breed</strong>: <span class="pet-breed">Golden Retriever</span><br/>
            <strong>Age</strong>: <span class="pet-age">3</span><br/>
            <strong>Location</strong>: <span class="pet-location">Warren, MI</span><br/><br/>
            <button class="btn btn-default btn-adopt" type="button" data-id="0">Adopt</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import config from '../assets/js/config.js'
/* eslint-disable no-undef */

var intervalQuery

export default {
  name: 'Pet',
  props: {
    msg: String
  },
  mounted () {
    this.init()
  },
  methods: {
    init: function () {
      // Load pets.
      var petsRow = $('#petsRow')
      var petTemplate = $('#petTemplate')

      for (let i in config) {
        petTemplate.find('.panel-title').text(config[i].name)
        petTemplate.find('img').attr('src', config[i].picture)
        petTemplate.find('.pet-breed').text(config[i].breed)
        petTemplate.find('.pet-age').text(config[i].age)
        petTemplate.find('.pet-location').text(config[i].location)
        petTemplate.find('.btn-adopt').attr('data-id', config[i].id)

        petsRow.append(petTemplate.html())
      }

      this.markAdopted()
      this.bindEvents()
    },
    bindEvents: function () {
      $(document).on('click', '.btn-adopt', this.handleAdopt)
    },
    handleAdopt: function (event) {
      event.preventDefault()

      var petId = parseInt($(event.target).data('id'))

      this.adoptPet(petId)
    },
    markAdopted: function () {
      var $pets = $('.panel-pet')

      let $pet = $($pets[0])


      for (let i = 0, len = $pets.length; i < len; i++) {
        let $pet = $($pets[i])
        let petId = $pet.find('.btn-adopt').attr('data-id')

        this.fetchPet(petId).then(pet => {
          if (pet) {
            $pet.find('button').text('Success').attr('disabled', true)
          }
        })
      }
    },
    fetchPet (petId) {
      var from = Account.NewAccount().getAddressString()
      var callArgs = [petId]

      var opt = {
        chainID: glob_chainID,
        from: from,
        to: DappAddress,
        value: 0,
        nonce: 0,
        gasPrice: 1000000,
        gasLimit: 2000000,
        contract: {
          function: 'get',
          args: JSON.stringify(callArgs)
        }
      }

      return neb.api.call(opt).then((res) => {
        var result = res.result
        // res is an object, res.result is a JSON string
        console.log('return of rpc call: ' + JSON.stringify(result))

        if (result === 'null') {
          return null
        } else {
          // if result is not null, then it should be "return value" or "error message"
          try {
            result = JSON.parse(result)
          } catch (err) {
            // result is the error message
          }

          if (!!result.petId) {
          // "return value"
            return result
          } else {
          // "error message"
            return null
          }
        }
      }).catch((err) => {
        console.error('error:' + err.message)
      })
    },
    adoptPet (petId) {
      var to = DappAddress
      var value = '0'
      var callFunction = 'adopt'
      var callArgs = [petId]

      var serialNumber

      serialNumber = nebPay.call(to, value, callFunction, JSON.stringify(callArgs), {
        listener: function (res) {
        // 设置listener, 处理交易返回信息
          console.log('response of push: ' + JSON.stringify(res))
        }
      })

      intervalQuery = setInterval(() => {
        this.fetchPayInfo(serialNumber, petId)
      }, 20000)
    },
    fetchPayInfo (serialNumber, petId) {
      nebPay.queryPayInfo(serialNumber)
        .then((resp) => {
          console.log('tx result: ' + resp)
          // resp is a JSON string
          var respObject = JSON.parse(resp)
          if (respObject.code === 0) {
            alert(`set petId:${petId} succeed!`)
            clearInterval(intervalQuery)

            this.markAdopted()
          }
        })
        .catch((err) => {
          console.error(err)
        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus">

</style>
