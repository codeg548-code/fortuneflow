<script>
  import { onMount } from 'svelte';
  import { api } from '../lib/api.js';
  import { loadUser, paymentConfigured, paymentRows, showToast, user } from '../lib/store.js';
  import IdentityCard from '../components/IdentityCard.svelte';

  let montant = '';
  let loading = false;
  let profile = null;

  onMount(async () => {
    profile = await loadUser();
  });

  $: carteConfiguree = paymentConfigured(profile || $user);
  $: rows = paymentRows(profile || $user);

  async function handleSubmit(event) {
    event.preventDefault();
    loading = true;
    try {
      await api.createDepot(montant);
      await loadUser();
      showToast('Votre dépôt a été soumis pour validation avec succès.', 'success');
      montant = '';
    } catch (error) {
      showToast(error.message, 'error');
    } finally {
      loading = false;
    }
  }
</script>

<div class="ff-page ff-page--form">
  <div class="ff-form-card">
    <h1 class="ff-form-title">Recharger mon compte</h1>
    <p class="ff-form-subtitle">Vos informations sont pré-remplies. Indiquez le montant.</p>

    {#if carteConfiguree}
      <IdentityCard
        icon="wallet"
        label="Carte de paiement enregistrée"
        {rows}
        linkUrl="#/compte"
        linkText="Modifier la carte →"
      />

      <form on:submit={handleSubmit}>
        <div class="ff-form-group">
          <label for="id_montant">Montant envoyé (FCFA)</label>
          <input
            type="number"
            id="id_montant"
            class="ff-input"
            placeholder="minimum: 2500"
            min="2500"
            step="100"
            bind:value={montant}
            required
          />
          <small class="ff-hint">Montant minimum de recharge : 2 500 FCFA.</small>
        </div>
        <button type="submit" class="ff-btn ff-btn-primary" disabled={loading}>
          <i class="fas fa-paper-plane"></i> {loading ? 'Envoi...' : 'Soumettre le dépôt'}
        </button>
      </form>
    {:else}
      <div class="ff-alert ff-alert--warning">
        <i class="fas fa-exclamation-triangle"></i>
        <div>
          <strong>Carte de paiement requise</strong>
          <p>Enregistrez d'abord votre moyen de paiement (MTN, Orange Money) dans Mon compte.</p>
        </div>
      </div>
      <a href="#/compte" class="ff-btn ff-btn-primary">
        <i class="fas fa-user-cog"></i> Configurer ma carte
      </a>
    {/if}
  </div>
</div>
