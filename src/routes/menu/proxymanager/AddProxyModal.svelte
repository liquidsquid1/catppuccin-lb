<script lang="ts">
    import Modal from "../common/modal/Modal.svelte";
    import IconTextInput from "../common/setting/IconTextInput.svelte";
    import SwitchSetting from "../common/setting/SwitchSetting.svelte";
    import ButtonSetting from "../common/setting/ButtonSetting.svelte";
    import {addProxy as addProxyRest} from "../../../integration/rest";
    import {listen} from "../../../integration/ws";

    export let visible: boolean;

    $: disabled = validateInput(requiresAuthentication, hostPort, username, password);
    $: {
        if (!requiresAuthentication) {
            username = "";
            password = "";
        }
    }

    let requiresAuthentication = false;
    let hostPort = "";
    let username = "";
    let password = "";
    let forwardAuthentication = false;
    let loading = false;

    function validateInput(requiresAuthentication: boolean, host: string, username: string, password: string): boolean {
        let valid = /.+:[0-9]+/.test(host);

        if (requiresAuthentication) {
            valid &&= username.length > 0 && password.length > 0;
        }

        return !valid;
    }

    async function addProxy() {
        if (disabled) {
            return;
        }
        const [host, port] = hostPort.split(":");

        loading = true;
        await addProxyRest(host, parseInt(port), username, password, forwardAuthentication);
    }

    listen("proxyAdditionResult", () => {
        loading = false;
        visible = false;
        cleanup();
    });

    function cleanup() {
        requiresAuthentication = false;
        hostPort = "";
        username = "";
        password = "";
        forwardAuthentication = false;
    }
</script>

<Modal title="Add Proxy" bind:visible={visible} on:close={cleanup}>
    <IconTextInput title="Host:Port" icon="server" pattern=".+:[0-9]+" bind:value={hostPort}/>
    <SwitchSetting title="Requires Authentication" bind:value={requiresAuthentication}/>
    {#if requiresAuthentication}
        <IconTextInput title="Username" icon="user" bind:value={username}/>
        <IconTextInput title="Password" icon="lock" type="password" bind:value={password}/>
    {/if}
    <SwitchSetting title="Forward Microsoft Authentication" bind:value={forwardAuthentication}/>
    <ButtonSetting title="Add Proxy" {disabled} on:click={addProxy} listenForEnter={true} {loading}/>
</Modal>
