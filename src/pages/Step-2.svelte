<h3>Adım: 2-3</h3>
<p>
  Veritabanı bilgilerini bilmiyorsanız sunucunuzu sağlayan kurum ile iletişime
  geçin.
</p>

<div
  class="alert alert-dismissible text-danger"
  id="error"
  style="display: none;">
  <button class="close" type="button" on:click="{dismissErrorBox}">
    <span aria-hidden="true">&times;</span>
  </button>
  {errorCode}
</div>

<form on:submit|preventDefault="{submit}">
  <div class="form-group">
    <nav
      class="nav nav-pills flex-nowrap text-lg-left text-center"
      id="databaseTabs"
      role="tablist">
      <a
        aria-controls="mysql"
        aria-selected="true"
        class="nav-link bg-lightprimary text-primary"
        data-toggle="tab"
        href="#mysql"
        id="mysql-tab"
        role="tab"
        title="Veritabanı Türü">
        MySQL veya MariaDB
      </a>
    </nav>
  </div>

  <div class="form-group tab-content">
    <div
      aria-labelledby="mysql-tab"
      class="tab-pane fade show active"
      id="mysql"
      role="tabpanel">
      <div class="row">
        <div class="col-6">
          <div class="form-group">
            <label for="databaseAddress">Veri Tabanı Adresi</label>
            <input
              class="form-control"
              id="databaseAddress"
              placeholder="localhost:80"
              bind:value="{$db.host}"
              type="text" />
          </div>
        </div>
        <div class="col-6">
          <div class="form-group">
            <label for="databaseName">Veri Tabanı Adı</label>
            <input
              class="form-control"
              id="databaseName"
              placeholder="pano"
              bind:value="{$db.dbName}"
              type="text" />
          </div>
        </div>
        <div class="w-100"></div>
        <div class="col-6">
          <div class="form-group">
            <label for="databaseUserName">Veri Tabanı Kullanıcı Adı</label>
            <input
              class="form-control"
              id="databaseUserName"
              placeholder="admin"
              bind:value="{$db.username}"
              type="text" />
          </div>
        </div>
        <div class="col-6">
          <div class="form-group">
            <label for="databaseUserPassword">Veri Tabanı Şifresi</label>
            <input
              class="form-control"
              id="databaseUserPassword"
              placeholder="****************"
              bind:value="{$db.password}"
              type="password" />
          </div>
        </div>
        <div class="col-12">
          <div class="form-group">
            <label for="databaseTablePrefix">Veri Tabanı Tablo Öneki</label>
            <input
              class="form-control"
              id="databaseTablePrefix"
              placeholder="pano_"
              bind:value="{$db.prefix}"
              type="text" />
          </div>
        </div>
      </div>
    </div>

    <div
      aria-labelledby="sqlite-tab"
      class="tab-pane fade show"
      id="sqlite"
      role="tabpanel">
      sqlite
    </div>
  </div>

  <hr />

  <div class="w-100 d-flex justify-content-end">
    <a
      href="javascript:void(0);"
      class="btn btn-link"
      role="button"
      on:click="{back}"
      class:disabled="{buttonsLoading}"
      disabled="{buttonsLoading}">Geri</a>
    <button
      type="submit"
      class="btn btn-primary"
      class:disabled="{buttonsLoading || nextButtonDisabled}"
      disabled="{buttonsLoading || nextButtonDisabled}"
      >Devam Et
    </button>
  </div>
</form>

<script>
  import jQuery from "jquery";

  import { onDestroy } from "svelte";
  import { nextStep, backStep, db } from "../Store";
  import { ApiUtil, NETWORK_ERROR } from "../util/api.util";

  let buttonsLoading = false;
  let nextButtonDisabled = true;
  let errorCode;

  function submit() {
    buttonsLoading = true;

    ApiUtil.post("setup/dbConnectionTest", {
      host: $db.host,
      dbName: $db.dbName,
      username: $db.username,
      password: $db.password,
    })
      .then((response) => {
        if (response.data.result === "ok") {
          next();
        } else if (response.data.result === "error") {
          const errorCode = response.data.error;

          showError(errorCode);
        } else showError(NETWORK_ERROR);
      })
      .catch(() => {
        showError(NETWORK_ERROR);
      });
  }

  function showError(error) {
    buttonsLoading = false;

    errorCode = error;

    jQuery("#error").fadeIn();
  }

  function dismissErrorBox() {
    jQuery("#error").fadeOut("slow");
  }

  function next() {
    buttonsLoading = true;

    nextStep({
      step: 2,
      host: $db.host,
      dbName: $db.dbName,
      username: $db.username,
      password: $db.password,
      prefix: $db.prefix,
    });
  }

  function back() {
    if (!buttonsLoading) {
      buttonsLoading = true;

      backStep({
        step: 2,
      });
    }
  }

  const dbUnsubscribe = db.subscribe(() => {
    checkForm();
  });

  onDestroy(dbUnsubscribe);

  function checkForm() {
    nextButtonDisabled = !(
      $db.host !== "" &&
      $db.dbName !== "" &&
      $db.username !== ""
    );
  }
</script>
