<script lang='ts'>
	import { applyAction, deserialize } from "$app/forms";
	import { validateSchema } from "$shared/lib/validation";
	import { Box } from "$shared/ui/Box";
	import { Form, FormCol, FormRow } from "$shared/ui/Form";
	import Button from "@smui/button";
	import Textfield from "@smui/textfield";
	import type { ComponentEvents } from "svelte";
	import { derived, writable } from "svelte/store";
	import type { ActionResult } from "@sveltejs/kit";
	import Link from "$shared/ui/Link/Link.svelte";
	import { UsersOAuth2GitHub, UsersOAuth2List } from "$features/users/OAuth2";
	import { PocketBaseAuthSchema } from "$shared/model";

    interface $$Props {
        class?:string
    }
    
    let className = ''
    export { className as class }
    const isErrorVisible = writable(false)
    const loginResult = writable<ActionResult | undefined>(undefined)

    const passwordAuthSchema = PocketBaseAuthSchema.withPassword

    const fields = writable(passwordAuthSchema.getDefault())

    const passwordAuthSchemaResult = derived([fields, isErrorVisible], ([$fields, $isErrorVisible]) => {
        const res = validateSchema(passwordAuthSchema, $fields)
        if (!$isErrorVisible) {
            res.errors = {}
        }
        return res
    })

    const handler = {
        async submit(event:ComponentEvents<Form>['submit']) {
            event.preventDefault()
            isErrorVisible.set(true)
            if ($passwordAuthSchemaResult.isError) return

            const form = event.currentTarget as HTMLFormElement
            
            const response = await fetch(form.action, {
                method: 'POST',
                body: new FormData(form),
            })

            const result = deserialize(await response.text())
            loginResult.set(result)
            await applyAction(result)
        }
    }
    
</script>
<Box class={`UsersAuthLogin ${className}`}>
    <Form method='POST' action='/users/login?/password' on:submit={handler.submit}>
        <UsersOAuth2List slot='header'>
            <UsersOAuth2GitHub/>
        </UsersOAuth2List>
        <FormCol>
            <FormRow>
                <Textfield invalid={!!$passwordAuthSchemaResult?.errors.identity} bind:value={$fields.identity} required variant='outlined' input$name='identity' type='text' label="Логин"/>
            </FormRow>
            <FormRow>
                <Textfield invalid={!!$passwordAuthSchemaResult?.errors.password} bind:value={$fields.password} required variant='outlined' input$name='password' type='password' label="Пароль"/>
            </FormRow>
        </FormCol>
        <svelte:fragment slot='button'>
            <Button variant='unelevated' disabled={$passwordAuthSchemaResult.isError}>
                {#if $loginResult?.type === 'failure'}
                    Упс, чет не то
                {:else}
                    Проверочка
                {/if}
            </Button>
        </svelte:fragment>
        <Link href="/users/signup" slot='links'>Регистрация</Link>
    </Form>
</Box>