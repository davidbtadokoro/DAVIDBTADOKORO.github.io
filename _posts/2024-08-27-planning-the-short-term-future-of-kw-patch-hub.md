---
layout: post
title: Planning the (short term) future of kw patch-hub
date: 2024-08-27 15:00:00 -0300
categories: [rust dev, software engeneering, lore]
tags: [rust, kw, kw patch-hub, lore]
---

[In the last post]({% post_url
2024-08-14-starting-collaborative-work-in-kw-patch-hub %}), I've talked about
how I teamed up with three really competent developers to help me with
[patch-hub](https://github.com/kworkflow/patch-hub) in this semester. This post
is an overall plan for our work together.

> This post will probably be updated as we reassess and modify the plan
{: .prompt-info }

<br>

## **Goals**

From my knowledge of the problem at hand (i.e., the huge overhead Linux
developers face when interacting with patches sent to
[lore.kernel.org](https://lore.kernel.org/)), my experience with the first
attempt to implement `patch-hub`, the requisites captured from veteran Linux
developers (which happen to also be maintainers), and the current state of the
project, I can think of these (short term) goals:

### Increase the project's robustness

As I focused on delivering a [_vertical
prototype_](https://en.wikipedia.org/wiki/Software_prototyping#Vertical_prototype)
of `patch-hub`, there are many (and I mean many) vulnerable points that can lead
to bad user experience in the best cases, and to horrible crashes in the worst
cases. Security, privacy, and protection should also be considered, but this
isn't a priority now. The print below is from one of the first issues of the
project, which illustrates the lack of robustness.

![unwrap issue]({{site.url}}/images/unwrap_issue.png)

### Add user documentation as well as codebase documentation

Basically, there isn't any user documentation to direct users on how to use
`patch-hub`, nor documentation on the codebase, so files, modules, functions,
and so on are undocumented. Below is a simple example of some undocumented
methods from a custom type named `LoreSession`, defined as a lib component
(hence, should more than anything be documented) at the application's core.

```rust
pub fn get_representative_patches_ids(self: &Self) -> &Vec<String> {
    &self.representative_patches_ids
}

pub fn get_processed_patch(self: &Self, message_id: &str) -> Option<&Patch> {
    self.processed_patches_map.get(message_id)
}

pub fn process_n_representative_patches<T: PatchFeedRequest>(self: &mut Self, lore_api_client: &T, n: u32) -> Result<(), FailedFeedRequest> {
    let mut patch_feed: PatchFeed;
    let mut processed_patches_ids: Vec<String>;

    while self.representative_patches_ids.len() < usize::try_from(n).unwrap() {
        match lore_api_client.request_patch_feed(&self.target_list, self.min_index) {
            Ok(feed_response_body) => patch_feed = from_str(&feed_response_body).unwrap(),
            Err(failed_feed_request) => return Err(failed_feed_request),
        }

        processed_patches_ids = self.process_patches(patch_feed);
        self.update_representative_patches(processed_patches_ids);

        self.min_index += LORE_PAGE_SIZE;
    }

    Ok(())
}
```

### Cover untested core components

Without getting into much detail, the UI of `patch-hub` has three components
that roughly resemble the components of the
[_Model-View-Controller_](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)
architectural design pattern. None of those components are currently being
tested. The snippet below illustrates all that is currently being covered in the
project. Note that the UI components are all defined in the project binary
crate, which amounts to 0 tests.

```
     Running unittests src/lib.rs (target/debug/deps/patch_hub-5f0f091074729111)

running 22 tests
test lore_session::tests::should_generate_patch_reply_template ... ok
test mailing_list::tests::can_deserialize_mailing_list ... ok
test lore_session::tests::test_split_patchset_invalid_cases ... ok
test mailing_list::tests::can_serialize_mailing_list ... ok
test mailing_list::tests::should_sort_mailing_list_vec ... ok
test lore_session::tests::should_split_patchset_without_cover_letter ... ok
test lore_session::tests::should_split_patchset_complete ... ok
test patch::tests::can_deserialize_patch_without_in_reply_to ... ok
test patch::tests::can_deserialize_patch_with_in_reply_to ... ok
test lore_session::tests::should_extract_git_reply_command_from_patch_html ... ok
test lore_session::tests::can_initialize_fresh_lore_session ... ok
test lore_session::tests::should_process_one_representative_patch ... ok
test lore_session::tests::should_process_multiple_representative_patches ... ok
test patch::tests::test_update_patch_metadata ... ok
test lore_session::tests::should_prepare_reply_patchset_with_reviewed_by ... ok
test lore_session::tests::should_get_local_git_signature ... ok
test lore_session::tests::should_process_available_lists ... ok
test lore_session::tests::should_fetch_all_available_lists ... ok
test lore_api_client::tests::blocking_client_can_request_valid_available_lists ... ok
test lore_api_client::tests::blocking_client_can_request_valid_patch_html ... ok
test lore_api_client::tests::blocking_client_should_detect_failed_patch_feed_request ... ok
test lore_api_client::tests::blocking_client_can_request_valid_patch_feed ... ok

test result: ok. 22 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 5.07s

     Running unittests src/main.rs (target/debug/deps/patch_hub-ca91d812441afe4b)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests patch_hub

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

### Refactor codebase to pay the technical debt

Files can be broken down into smaller files for better modularity and
organization, and components can be better encapsulated and decoupled. The
technical debt isn't dismissable, and the code snippet below shows how the whole
_Controller_ component is defined in `src/main.rs` in a single function (don't
bother trying to visualize it; the mess illustrates how much we need
refactoring).

```rust
fn run_app<B: Backend>(terminal: &mut Terminal<B>, app: &mut App) -> color_eyre::Result<()> {
    loop {
        terminal.draw(|f| draw_ui(f, &app))?;

        match app.current_screen {
            CurrentScreen::MailingListSelection => {
                if app.mailing_list_selection_state.mailing_lists.is_empty() {
                    app.mailing_list_selection_state.refresh_available_mailing_lists()?;
                }
            },
            CurrentScreen::BookmarkedPatchsets => {
                if app.bookmarked_patchsets_state.bookmarked_patchsets.is_empty() {
                    app.set_current_screen(CurrentScreen::MailingListSelection);
                }
            },
            _ => {},
        }

        if event::poll(std::time::Duration::from_millis(16))? {
            if let Event::Key(key) = event::read()? {
                if key.kind == event::KeyEventKind::Release {
                    // Skip events that are not KeyEventKind::Press
                    continue;
                }
                match app.current_screen {
                    CurrentScreen::MailingListSelection if key.kind == KeyEventKind::Press => {
                        match key.code {
                            KeyCode::Enter => {
                                if app.mailing_list_selection_state.has_valid_target_list() {
                                    app.init_latest_patchsets_state();
                                    app.latest_patchsets_state.as_mut().unwrap().fetch_current_page()?;
                                    app.mailing_list_selection_state.clear_target_list();
                                    app.set_current_screen(CurrentScreen::LatestPatchsets);
                                }
                            }
                            KeyCode::F(5) => {
                                app.mailing_list_selection_state.refresh_available_mailing_lists()?;
                            }
                            KeyCode::F(1) => {
                                if !app.bookmarked_patchsets_state.bookmarked_patchsets.is_empty() {
                                    app.mailing_list_selection_state.clear_target_list();
                                    app.set_current_screen(CurrentScreen::BookmarkedPatchsets);
                                }
                            }
                            KeyCode::Backspace => {
                                app.mailing_list_selection_state.remove_last_target_list_char();
                            }
                            KeyCode::Esc => {
                                return Ok(());
                            }
                            KeyCode::Char(ch) => {
                                app.mailing_list_selection_state.push_char_to_target_list(ch);
                            }
                            KeyCode::Down => {
                                app.mailing_list_selection_state.highlight_below_list();
                            },
                            KeyCode::Up => {
                                app.mailing_list_selection_state.highlight_above_list();
                            },
                            _ => {}
                        }
                    },
                    CurrentScreen::LatestPatchsets if key.kind == KeyEventKind::Press => {
                        match key.code {
                            KeyCode::Esc => {
                                app.reset_latest_patchsets_state();
                                app.set_current_screen(CurrentScreen::MailingListSelection);
                            },
                            KeyCode::Char('j') | KeyCode::Down => {
                                app.latest_patchsets_state.as_mut().unwrap().select_below_patchset();
                            },
                            KeyCode::Char('k') | KeyCode::Up => {
                                app.latest_patchsets_state.as_mut().unwrap().select_above_patchset();
                            },
                            KeyCode::Char('l') | KeyCode::Right => {
                                app.latest_patchsets_state.as_mut().unwrap().increment_page();
                                app.latest_patchsets_state.as_mut().unwrap().fetch_current_page()?;
                            },
                            KeyCode::Char('h') | KeyCode::Left => {
                                app.latest_patchsets_state.as_mut().unwrap().decrement_page();
                            },
                            KeyCode::Enter => {
                                app.init_patchset_details_and_actions_state(CurrentScreen::LatestPatchsets)?;
                                app.set_current_screen(CurrentScreen::PatchsetDetails);
                            },
                            _ => {}
                        }
                    },
                    CurrentScreen::BookmarkedPatchsets if key.kind == KeyEventKind::Press => {
                        match key.code {
                            KeyCode::Esc => {
                                app.bookmarked_patchsets_state.patchset_index = 0;
                                app.set_current_screen(CurrentScreen::MailingListSelection);
                            },
                            KeyCode::Char('j') | KeyCode::Down => {
                                app.bookmarked_patchsets_state.select_below_patchset();
                            },
                            KeyCode::Char('k') | KeyCode::Up => {
                                app.bookmarked_patchsets_state.select_above_patchset();
                            },
                            KeyCode::Enter => {
                                app.init_patchset_details_and_actions_state(CurrentScreen::BookmarkedPatchsets)?;
                                app.set_current_screen(CurrentScreen::PatchsetDetails);
                            },
                            _ => {}
                        }
                    },
                    CurrentScreen::PatchsetDetails if key.kind == KeyEventKind::Press => {
                        match key.code {
                            KeyCode::Esc => {
                                app.set_current_screen(
                                    app.patchset_details_and_actions_state.as_ref().unwrap().last_screen.clone()
                                );
                                app.reset_patchset_details_and_actions_state();
                            },
                            KeyCode::Char('j') | KeyCode::Down => {
                                app.patchset_details_and_actions_state.as_mut().unwrap().preview_scroll_down();
                            },
                            KeyCode::Char('k') | KeyCode::Up => {
                                app.patchset_details_and_actions_state.as_mut().unwrap().preview_scroll_up();
                            },
                            KeyCode::Char('n') => {
                                app.patchset_details_and_actions_state.as_mut().unwrap().preview_next_patch();
                            },
                            KeyCode::Char('p') => {
                                app.patchset_details_and_actions_state.as_mut().unwrap().preview_previous_patch();
                            },
                            KeyCode::Char('b') => {
                                app.patchset_details_and_actions_state.as_mut().unwrap().toggle_bookmark_action();
                            },
                            KeyCode::Char('r') => {
                                app.patchset_details_and_actions_state.as_mut().unwrap().toggle_reply_with_reviewed_by_action();
                            },
                            KeyCode::Enter => {
                                if app.patchset_details_and_actions_state.as_ref().unwrap().actions_require_user_io() {
                                    utils::setup_user_io(terminal)?;
                                    app.consolidate_patchset_actions()?;
                                    println!("\nPress ENTER continue...");
                                    loop {
                                        if let Event::Key(key) = event::read()? {
                                            if key.kind == event::KeyEventKind::Press && key.code == KeyCode::Enter {
                                                break; 
                                            }
                                        }
                                    }
                                    utils::teardown_user_io(terminal)?;
                                } else {
                                    app.consolidate_patchset_actions()?;
                                }
                                app.set_current_screen(CurrentScreen::PatchsetDetails);
                            },
                            _ => {}
                        }
                    },
                    _ => {},
                }
            }
        }
    }
}
```

### Add remaining actions on patchsets

One of the core services `patch-hub` aims to provide is running actions on
individual patchsets (and their patches). This includes:

1. Replying the patches in the series with git tags (`Reviewed-by`, `Acked-by`, `Tested-by`, and so on)
2. Replying the patches in the series with inline reviewing (done inside or outside `patch-hub`)
3. Applying the series into a target branch of a Linux kernel tree to ensure there aren't merge conflicts.
4. Building a custom Linux kernel for each patch on the series to ensure no patches break compilation.

The print below shows the only two options implemented, which are
(un)bookmarking a patchset and replying to the entire series with the
`Reviewed-by` tag.

![current_patch_hub_actions]({{site.url}}/images/current_patch_hub_actions.png)

### EXTRAS

There are many more opportunities to enhance the project. Even though the above
items are more than a handful, here are some extra goals, just to list a few:

1. Package `patch-hub` for Debian, Arch, and Fedora
2. Enhance UI
3. Enhance UX
4. Allow scheduling of actions

## Conclusion

I am deeply invested and anxious for the future of `patch-hub`. Even more so
considering the privilege of working with such a capable team.

There is a lot of work to be done, and if we can make it to roughly 80% of the
previously mentioned goals, I will be more than happy.

Sure, as time passes and software projects evolve, goals change, unforeseen
problems arise, and life happens. However, I am confident we can get `patch-hub`
to a marvelous state!
