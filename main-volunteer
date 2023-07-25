// Setting window.skipTfcJs will skip this script and run the next one.
if (window.skipTfcJsTest) {
  window.skipTfcJsTest = false;
  throw `Skipping ${document.currentScript.src}`;
} else {
  console.log(`Executing ${document.currentScript.src}`);
}

// Naming convention: to avoid confusion between DOM elements and jQuery
// objects, all names referring to jQuery objects end in "Jq".

ACCESS_GUARD_HOSTNAME = 'oj9r1tx3hj.execute-api.us-east-1.amazonaws.com';
ACCESS_GUARD_STAGING_HOSTNAME = 'phmlmlepyl.execute-api.us-east-1.amazonaws.com';
RECAPTCHA_SITEKEY = '6Ld-CSQeAAAAAJWio0E7S-Ut9qLi1yI0akx46xWz';
RECRUITER_SIGNUP_URL = "https://api.techforcampaigns.io/volunteer/respond_api";

// The length of time that our Access Guard will honour the JWT.  This
// constant has no authorization impact; it merely ensures that the UI
// doesn't appear to be logged in after the token has expired.  For the UI
// to be correct, this should match the TTL constant in the Access Guard.

APPARENT_JWT_TTL = 24 * 3600;

// Autocompletions for the "Skill set" field.

SKILL_AUTOCOMPLETIONS = [
    'Analytics',
    'Backend Engineering',
    'Business Development / Partnerships',
    'Content Strategy & Copywriting',
    'Data Science',
    'Email Design',
    'Email Marketing',
    'Frontend Engineering',
    'Full-Stack Engineering',
    'General Management (Engineering)',
    'General Management (Marketing)',
    'Graphic Design',
    'Mobile Engineering',
    'Motion Graphic Design',
    'Paid / Growth Marketing',
    'Product Management',
    'Product Marketing',
    'Project Management / Operations',
    'Public Relations',
    'Sales',
    'Security Engineering',
    'Social Media Marketing',
    'UI / UX Design',
    'Video Editing',
    'Video Production',
];

// If the URL before redirection has query parameters like ?utm_source=...,
// then the Webflow redirection will yield a URL with two question marks in it.
// The part after the first question mark is the slug, and the part after the
// second question mark contains the original query parameters.
match = window.location.search.match(/\??([^?]*)(\?.*)?/);
slug = match[1] || '';
query = match[2] || '';

// Globally available boolean flags that also appear as attributes on <body>.
flags = (() => {
  const values = {};
  return new Proxy(values, {
    set: (target, name, value) => {
      let newValue = value ? 1 : 0;
      if (newValue !== values[name]) {
        $('body').attr('x-' + name, (values[name] = value ? 1 : 0));
        renderAllTemplates();
      }
    }
  });
})();

// Mark the body x-ready when all concurrent rendering operations are done.
rendersInProgress = (() => {
  let count = 0;
  return {
    inc: () => count++,
    dec: () => (--count === 0) && handleRenderComplete()
  };
})();

// Schedule a rendering of all templates to take place asynchronously.
renderAllTemplates = () => {
  window.setTimeout(() => {
    rendersInProgress.inc();
    eachNode($('[x-template]'), renderTemplate);
    rendersInProgress.dec();
  }, 1);
};

// jQuery's $.each() and .each() and .map() have type signatures that trip me
// up every time.  These are a little clearer: the callback function is always
// called with the array item as its one argument; eachJq and mapJq pass each
// item as a jQuery object, whereas eachNode passes each item as a DOM node.
// Unlike jQuery's .map(), mapJq always returns an array of the results.
eachJq = (jq, fn) => $.each(jq, (index, node) => fn($(node)));
eachNode = (jq, fn) => jq.each((index, node) => fn(node));
mapJq = (jq, fn) => jq.map((index, node) => fn($(node))).toArray();

// Functions for use in template expressions.

replaceTfcCss = (url) => $('link[id="tfc-css"]').attr('href', url);

toKebabCase = (name) => name.trim().replace(/\W+/g, '-').toLowerCase();

sortProjects = (projects) => projects.sort(
  (a, b) => a.recruiting_start_date.localeCompare(b.recruiting_start_date)
);

getSelectedRole = (roles) => {
  let roleName = getValue($('.role-radio-button:checked'));
  for (const role of roles) {
    if (role.role === roleName) return role;
  }
};

fixApparentRfvUrl = () => {
  let path = window.location.pathname;
  if (!path.endsWith('/' + slug)) path += '/' + slug;

  // We append 'query' to preserve any original params such as ?utm_source=...
  history.replaceState(null, null, path + query);
};

dismiss = (jq) => {
  let dis = jq.closest('[x-dismissable]');
  if (dis.length) {
    Cookies.set('dismissed.' + dis.attr('id'), 1, {expires: 1 /* day */});
    dis.attr('x-dismissed', 1);
  }
};

reveal = (jq) => {
  jq.attr('x-revealed', 1);
  history.pushState({}, '', '#' + jq.attr('x-id'));
};

unreveal = (jq) => {
  jq.attr('x-revealed', 0);
  history.back();
};

// Webflow refuses to let us set a custom "value" attribute (complaining
// that it is "reserved") so sometimes we need to use x-value instead.
// This overrides val() because val() always returns "on" for checkboxes;
// and val() overrides the "value" attribute because for input fields, we
// want the contents of the field, not the initial value in the attribute.
getValue = (jq) => jq.attr('x-value') || jq.val() || jq.attr('value');

// Triggers a re-render after intervalMs milliseconds.  This is meant to be
// invoked by x-eval="reloadDraftRfv(...)" in the template.  When the re-render
// occurs, it will evaluate the x-eval attribute again, invoking this function,
// which will schedule another re-render, and so on -- causing the page to be
// repeatedly re-rendered at intervals of intervalMs milliseconds.
reloadDraftRfv = (project, intervalMs) => {
  updateDraftTimestamp($('[x-id=draft-timestamp]'), getMaxTimestamp(project));
  setTimeout(renderAllTemplates, intervalMs);
};

// Validators return either the valid normalized value, or null if invalid.

validateEmail = (value) => {
  let address = value.trim().toLowerCase();
  return address.match(/^[-\w.+]+@[-\w_.]+\.[a-z][a-z]+$/) ? address : null;
};

validateUrl = (value) => {
  let url = value.trim();
  return url.match(/^(https?:\/\/)?[-\w_.]+\.[a-zA-Z]{2,}(\/\S*)?/) ? url : null;
};

validatePhone = (value) => {
  let digits = value.replace(/[^0-9]/g, '');
  let match = digits.match(/^1?([2-9]\d\d)(\d{3})(\d{4})$/);
  return match ? `(${match[1]}) ${match[2]}-${match[3]}` : null;
};

validateZip = (value) => {
  let digits = value.replace(/[\s()-]/g, '');
  if (digits.length === 5) return digits;
  if (digits.length === 9) return digits.substr(0, 5) + '-' + digits.substr(5);
};

// Input filters return the filtered input value (never null).

filterDigitsOnly = (value) => value.replace(/[^0-9() +-]/g, '');

// Control visibility based on the URL fragment.

revealByFragment = () => {
  let fragment = window.location.hash.replace('#', '');
  let jq = $(`[x-id="${fragment}"][x-revealed]`).eq(0);
  if (jq.length) reveal(jq);
};

showHideByFragment = () => {
  $('[x-shown-if-fragment]').hide();  // default to hidden
  $('[x-hidden-if-fragment]').show();  // default to shown
  let fragment = window.location.hash.replace('#', '');
  if (fragment) {
    $(`[x-shown-if-fragment="${fragment}"]`).show();
    $(`[x-hidden-if-fragment="${fragment}"]`).hide();
  }
};

// Sets up a handler so that the navbar, if transparent, becomes opaque when
// the user scrolls down or the navigation menu is open.
setupNavbarToggler = () => {
  let transparentOk = $('body').attr('x-transparent-navbar');
  let menuButton = $('[x-id=menu-button]');
  let updateNavbarMode = () => {
    let forceOpaque = window.pageYOffset >= 6 || menuButton.hasClass('w--open');
    $('body').attr('x-transparent-navbar', transparentOk && !forceOpaque ? 1 : 0);
  }

  // Update the navbar if the navigation menu's attributes change.
  eachNode(menuButton, node =>
    new MutationObserver(updateNavbarMode).observe(node, {'attributes': true})
  );

  // Update the navbar when the window is scrolled.
  $(document).on('scroll', () => requestAnimationFrame(updateNavbarMode));
};

// Sets up reCAPTCHA on all forms.
setupAllRecaptchaForms = () => {
  if (window.isReflectTest) {
    // This turns off reCAPTCHA on the client side when we are running an
    // automated test with Reflect.  Merely setting this flag doesn't expose
    // us to spam, though, because form submissions go through server-side
    // validation as well (using the "Enable reCAPTCHA" setting in Webflow).
    return;
  }
  //$.getScript('https://www.google.com/recaptcha/api.js').done(() =>
  //  grecaptcha.ready(() => eachJq($('form'), setupRecaptcha))
  //);
  $.ajax({
    async: false,
    url: "https://www.google.com/recaptcha/api.js",
    dataType: "script",
    crossDomain: "true"
  }).done(() =>
    grecaptcha.ready(() => eachJq($('form'), setupRecaptcha))
  ).fail(function() {
    return;
  });
};

// Given a form, adds a hidden input field containing the site's domain name
// to help us distinguish production from staging.
setupDomainFields = () => {
  eachJq($('form'), formJq => {
    let fieldJq = formJq.find('input[name=domain]');
    if (fieldJq.length === 0) {
      fieldJq = $('<input type="hidden" name="domain">').appendTo(formJq);
    }
    fieldJq.val(window.location.hostname);
  });
};

// Patches the Webflow form submission handler to run reCAPTCHA before
// submission (see https://developers.google.com/recaptcha/docs/invisible).
// The usual Webflow form validation will take place, and then if the user
// passes reCAPTCHA, the Webflow form submission logic will proceed.
// The flow of control goes: submit form -> validation -> data.handler
// -> executeRecaptcha -> handleRecaptchaResult -> data.webflowHandler.
setupRecaptcha = (formJq) => {
  let originalHandler = null;
  if (formJq.find('input[name=g-recaptcha-response]').length === 0) {
    $('<input type="hidden" name="g-recaptcha-response">').appendTo(formJq);
  }

  // Creates the invisible reCAPTCHA prompt, yielding a unique ID for it.
  let recaptchaId = grecaptcha.render($('<div>').appendTo(formJq)[0], {
    callback: response => {
      formJq.find('[name=g-recaptcha-response]').val(response);
      originalHandler(formJq.data('.w-form'));
    },
    sitekey: RECAPTCHA_SITEKEY,
    size: 'invisible'
  });

  // Prevents normal form submission and invokes reCAPTCHA instead.
  let executeRecaptcha = (data) => {
    data.evt.preventDefault();
    grecaptcha.execute(recaptchaId);
  }

  // Upon initial page load, formJq.data('.w-form') doesn't exist yet.  So, we
  // wait until submission time to set up executeRecaptcha to intercept it.
  formJq.find('input[type=submit]').click(() => {
    let data = formJq.data('.w-form');  // Webflow puts the form data here
    if (!originalHandler) originalHandler = data.handler;
    data.handler = executeRecaptcha;
  });
};

// Evaluates a JavaScript expression in the given context.  For example,
// evalJs(null, 'a.b + c', {a: {b: 3}, c: 6}) will return 9.
evalJs = (thisJq, expr, context) => {
  context = context || {};
  const names = Object.keys(context);
  const values = Object.values(context);
  try {
    const func = Function.call(null, ...names, 'return ' + expr);
    return func.call(thisJq, ...values);
  } catch (e) {
    console.log('Could not evaluate', expr, 'in', context, '' + e);
  }
};

// Renders a string template by replacing any expressions enclosed in
// curly braces with their values as evaluated in the given context.
renderString = (thisJq, template, context) => (template || '').replace(
  /\{(.*?)\}/g,
  (all, expr) => evalJs(thisJq, expr, context) || ''
);

originalNodeTexts = new WeakMap();
lastGroupId = 1;

// Renders a DOM template by updating text contents, classes, and attributes
// in place under the given root node.  The original text content is saved;
// calling renderTemplate on the same node again will process the original
// text as a template again, rather than treating the output as a new template.
//
// Attributes with special behaviour are interpreted in this order:
//   x-fetch-<name>="url" fetches JSON data from a URL and assigns a name to it
//   x-for="var" x-in="array" renders a clone of the element for each array item
//   x-if="condition" hides and skips the element if the JS condition is false
//   x-eval="expr" evaluates a JS expression
//   x-let-<name>="expr" assigns a name to the value of a JS expression
//   x-attr-<name>="template" sets an attribute by rendering a string template
//   x-attr-<name>-if="condition" sets an attribute to 0 or 1 given a condition
//   x-class="template" adds a class by rendering a string template
//   x-class-<name>-if="condition" adds or removes a class given a condition
renderTemplate = (element, context, skipFetch) => {
  let elementJq = $(element);
  let eval = (expr, context) => evalJs(elementJq, expr, context);
  let render = (value, context) => renderString(elementJq, value, context);

  rendersInProgress.inc();
  if (!skipFetch) {
    const fetches = [];
    for (let {name, value} of element.attributes) {
      let match = name.match(/^x-fetch-(.+)$/);
      if (match) fetches.push([match[1], value]);
    }
    if (fetches.length) {
      const newContext = {...context};
      const promises = [];
      for (let [name, value] of fetches) {
        let url = render(value, context);
        // Automatically send the JWT when fetching from our Access Guard.
        let hostname = new URL(url).hostname;
        let headers = (
          hostname === ACCESS_GUARD_HOSTNAME ||
          hostname === ACCESS_GUARD_STAGING_HOSTNAME
        ) ? {Authorization: 'Bearer ' + getJwt()} : {};
        promises.push(fetch(url, {headers: headers}).then(response =>
          response.json().then(data => newContext[name] = data)
        ).catch(console.log));
      }
      // Let the fetches run asynchronously and skip the rest of this subtree;
      // then when all the fetches are done, continue rendering this subtree.
      Promise.all(promises).then(() => {
        renderTemplate(element, newContext, true);
        rendersInProgress.dec();
      });
      return;
    }
  }

  if (elementJq.attr('x-for')) {
    // Remove previously rendered clones; new clones will get a new group ID.
    elementJq.nextAll(`[x-group-id=${elementJq.attr('x-group-id')}]`).remove();
    let newGroupId = ++lastGroupId;
    elementJq.attr('x-group-id', newGroupId);

    // Create a clone for each array item, render it, and append it.
    let loopVar = elementJq.attr('x-for');
    let lastJq = elementJq;
    for (let item of eval(elementJq.attr('x-in'), context) || []) {
      let newCloneJq = elementJq.clone().removeAttr('x-for').attr('x-clone', 1);
      lastJq = renderTemplate(
        newCloneJq[0], {...context, [loopVar]: item}
      ).insertAfter(lastJq);
    }

  } else if (!eval(elementJq.attr('x-if') || 1, context)) {
    elementJq.hide();  // hide and skip descendants

  } else {
    // Okay, we are keeping this node; render its attributes.
    eval(elementJq.attr('x-eval') || 1, context);

    for (let {name, value} of element.attributes) {
      let match = name.match(/^x-let-(.+)$/);
      if (match) context = {...context, [match[1]]: eval(value, context)};
    }

    for (let {name, value} of element.attributes) {
      let match = name.match(/^x-attr-(.+?)(-if)?$/);
      if (match?.[2]) elementJq.attr(match[1], eval(value, context) ? 1 : 0);
      else if (match) elementJq.attr(match[1], render(value, context) || null);

      match = name.match(/^x-class-(.+)-if$/);
      if (match) elementJq.toggleClass(match[1], eval(value, context) || false);
      if (name === 'x-class') elementJq.addClass(render(value, context));
    }

    // Finally, render its children.
    if (element.tagName !== 'SCRIPT' && element.tagName !== 'STYLE') {
      eachNode(elementJq.contents(), node => {
        if (node.nodeType === Node.TEXT_NODE) {
          let value = originalNodeTexts.get(node) || node.nodeValue;
          originalNodeTexts.set(node, value);
          node.nodeValue = render(value, context);
        }
        // Caution!  We are iterating over child nodes while possibly also
        // inserting freshly rendered clones in front of the iterator, so we
        // must be careful to skip re-rendering the already rendered clones.
        if (node.tagName && !node.hasAttribute('x-clone')) {
          renderTemplate(node, context);
        }
      });
    }
  }

  rendersInProgress.dec();
  return elementJq;
};

// Moves all <title> and <meta> elements found in the page into the <head>
// element, replacing their counterparts.  This allows us to update the title,
// for example, by rendering a <title> tag in the body of the page using a
// template.  Browsers tend to respect the first <title> tag on the page,
// so we need to overwrite it for a new title to take effect.
gatherHeadElements = () => {
  let titleJq = $('title').last();
  let metasJq = {};
  eachJq($('meta'), jq => metasJq[jq.attr('name') || jq.attr('property')] = jq);
  $('title, meta').remove();
  $('head').prepend(Object.values(metasJq)).prepend(titleJq);
};

lastTimestamp = 0;

updateDraftTimestamp = (jq, timestamp) => {
  let updated = new Date(timestamp);
  let date = updated.toDateString();
  date = date.replace(/^\w+ /, '').replace(/ \d\d\d\d$/, '').replace(' 0', ' ');
  let time = updated.toTimeString().substring(0, 5);
  let age = new Date().getTime() - timestamp;
  let seconds = Math.floor(age/1000);
  let minutes = Math.floor(seconds/60);
  let hours = Math.floor(minutes/60);
  let days = Math.floor(hours/24);
  let interval =
    days >= 730 ? Math.floor(days/365) + ' years ago' :
    days >= 90 ? Math.floor(days/30) + ' months ago' :
    days >= 28 ? Math.floor(days/7) + ' weeks ago' :
    days >= 2 ? days + ' days ago' :
    hours >= 2 ? hours + ' hours ago' :
    minutes >= 2 ? minutes + ' minutes ago' :
    seconds >= 5 ? (seconds - (seconds % 5)) + ' seconds ago' :
    'just now';
  jq.text(`DRAFT: Updated ${interval} (${date}, ${time})`);
  jq.attr('x-flash', timestamp > lastTimestamp ? 'orange' : 'normal');
  lastTimestamp = timestamp;
};

// Constructs the filterByFormula query parameter for the Airtable recipe API.
// Formulas for the Airtable API follow the syntax:
// https://support.airtable.com/hc/en-us/articles/203255215-Formula-Field-Reference
// The Recipes API:
// https://airtable.com/appQaQfATHU5TyMVV/api/docs#javascript/table:recipe%20book:list
// TAKES: filters (object): includes channel (string), cycle (string), platforms (array) and objectives (array)
// RETURNS: a URL query parameter string, beginning with "filterByFormula="
getRecipeFilterParams = () => {
  let filters = {};
  const platformSelection = getValue($('#platforms-filter option:selected'));
  filters.platforms = platformSelection ? [platformSelection] : [];
  const objectiveSelection = getValue($('#objectives-filter option:selected'));
  filters.objectives = objectiveSelection ? [objectiveSelection] : [];
  filters.channel = getValue($('#channel-filter option:selected'));
  filters.cycle = getValue($('#cycle-filter option:selected'));

  let activeFilters = [];
  if (filters.channel) {
    // Example (unencoded) formula '{Medium}="Digital Ads"'
    const channelFormula = '{Channel}="' + filters.channel + '"';
    activeFilters.push(channelFormula);
  }
  if (filters.cycle) {
    // Example (unencoded) formula '{Year}="2021"'
    const cycleFormula = '{Year}="' + filters.cycle + '"';
    activeFilters.push(cycleFormula);
  }
  if (filters.platforms && filters.platforms.length) {
    // If we want to enable mlutiple select, this will be an array with length >1
    // For now the length is always 1 and the OR is superfluous.
    // Example: 'OR(FIND("Facebook",{Platform}),FIND("GDN",{Platform}))'
    const platformFormula = 'OR(' +
       filters.platforms.map(x=>`FIND("${x}",{Platform})`).join(',') +
       ')';
    activeFilters.push(platformFormula);
  }
  if (filters.objectives && filters.objectives.length) {
    // Example: 'OR(FIND("Fundraising",{Objective}),FIND("Persuasion",{Objective}))'
    const objectiveFormula = 'OR(' +
      filters.objectives.map(x=>`FIND("${x}",{Objective})`).join(',') +
      ')';
    activeFilters.push(objectiveFormula);
  }
  if (activeFilters.length > 0) {
    return 'filterByFormula=' + encodeURIComponent(
      'AND(' + activeFilters.join(',') + ')');
  }
  return '';
};

// Searches for numerical "timestamp" fields among all the properties of
// the given object and its descendants, and returns the largest one.
getMaxTimestamp = (struct) => {
  if (struct == null || typeof struct !== 'object') return 0;
  const timestamps = [struct.timestamp || 0];
  for (const item of Array.from(struct)) timestamps.push(getMaxTimestamp(item));
  for (const key in struct) timestamps.push(getMaxTimestamp(struct[key]));
  return Math.max.apply(null, timestamps);
};

// Adds an option (val) to a select filter
addFilterOption = (selectJq, val) =>
  selectJq.append($("<option></option>").val(val).text(val));

initFilterOptions = (recipes) => {
  if (flags['filters-ready']) return;
  flags['filters-ready'] = 1;

  // Sets will automatically only add unique values
  let allChannels = new Set();
  let allCycles = new Set();
  let allObjectives = new Set();
  let allPlatforms = new Set();

  // Collect values from Airtable recipe records
  for (const recipe of recipes) {
    allChannels.add(recipe.fields.Channel);
    allCycles.add(recipe.fields.Year);
    recipe.fields.Platform.forEach(x => allPlatforms.add(x))
    recipe.fields.Objective.forEach(x => allObjectives.add(x))
  }
  // Alphabetical (or numerical) sorting is more user friendly
  const sortedChannels = Array.from(allChannels).sort();
  const sortedCycles = Array.from(allCycles).sort().reverse();
  const sortedPlatforms = Array.from(allPlatforms).sort();
  const sortedObjectives = Array.from(allObjectives).sort();
  // Remove filter select options except for first (the blank one)
  eachJq($(".recipe-filter"), jq => jq.find('option').slice(1).remove());
  // Append new options to filters (in sorted order)
  sortedChannels.forEach(val => addFilterOption($("#channel-filter"), val));
  sortedCycles.forEach(val => addFilterOption($("#cycle-filter"), val));
  sortedPlatforms.forEach(val => addFilterOption($("#platforms-filter"), val));
  sortedObjectives.forEach(val => addFilterOption($("#objectives-filter"), val));

  $('.recipe-filter').change(renderAllTemplates);
};

postFormAsJson = (form, url, contentType) => {
  let data = new FormData(form);
  let body = {};
  for (let key of data.keys()) {
    body[key] = data.get(key);
  }
  return fetch(url, {
    method: 'POST',
    headers: contentType ? {'Content-Type': contentType} : {},
    body: JSON.stringify(body)
  });
};

// Sets up any form marked with x-id="rfv-form" so that it submits a
// volunteer application form to Recruiter.
setupRfvForm = () => {
  let formJq = $('[x-id=rfv-form]');
  if (formJq.length) {
    let submitButtonJq = formJq.find('input[type=submit]');
    formJq.submit(async event => {
      formJq.attr('x-loading', 1);
      submitButtonJq.attr('disabled', true);
      event.preventDefault();

      // Send the form values to Recruiter instead of Webflow.
      const response = await postFormAsJson(
        formJq[0], RECRUITER_SIGNUP_URL, 'application/json');

      formJq.attr('x-loading', 0);
      formJq.addClass(response.ok ? 'success' : 'error');
      if (!response.ok) {
        submitButtonJq.attr('disabled', false); // let the user try again
      }
    });
  }
};

// Sets up an "option set", given a container of options, checkboxes, or radio
// buttons, of which one can be the "other" option (marked x-other-option="1")
// that reveals a text field (marked x-other-field="1").  The input field
// marked x-selected-values-field="1" is continuously updated with the
// comma-separated values of the selected options, and the input field
// marked x-selected-groups-field="1" is continously updated with the
// comma-separated values of any optgroups containing the selected options.
// The container itself can also have a x-max-selections attribute, which
// restricts the number of items that the user is allowed to select.
createOptionSet = (rootJq) => {
  const selectedValuesFieldJq = rootJq.find('[x-selected-values-field]');
  const selectedGroupsFieldJq = rootJq.find('[x-selected-groups-field]');
  const otherOptionsJq = rootJq.find('[x-other-option]');
  const otherFieldJq = rootJq.find('[x-other-field]');

  const getOptions = () => rootJq.find(
    'input[type=checkbox], input[type=radio], option');
  const getSelected = () => rootJq.find(
    'input:checked, option:selected');
  const getValues = () => mapJq(getSelected(), getValue);
  const getGroupValues = () => mapJq(
    getSelected().closest('optgroup'), getValue
  );

  const maxSelections = rootJq.attr('x-max-selections') - 0;
  if (maxSelections) getOptions().change(event => {
    // We receive the click event AFTER the checkbox has been toggled.
    if (getValues().length > maxSelections) {
      event.target.checked = false;
      alert(`Please choose at most ${maxSelections} items.`);
    }
  });

  otherFieldJq.css('visibility', 'hidden');
  getOptions().add(rootJq.find('select')).change(() => {
    selectedValuesFieldJq.val(getValues().join(','));
    selectedGroupsFieldJq.val(getGroupValues().join(','));
    otherFieldJq.css(
      'visibility', getSelected().is(otherOptionsJq) ? 'visible' : 'hidden');
  });

  return {
    getValues: getValues,
    finalize: () => {  // remove the input elements we don't want to submit
      getOptions().remove();
      otherFieldJq.remove();
    }
  };
};

// Sets up forms marked with x-redirect-to="url" to redirect to a new URL
// after successful submission.  Any input fields with x-redirect-param="name"
// will have their values included in a query string attached to the new URL.
setupFormRedirection = () => {
  eachJq($('form[x-redirect-to]'), formJq => {
    let match = formJq.attr('x-redirect-to').match(/([^#]*)(#?.*)/);
    let [all, path, fragment] = match;
    let originalHandler = null;

    let submitAndRedirect = (data) => {
      let params = {};
      eachJq(formJq.find('[x-redirect-param]'), jq => {
        params[jq.attr('x-redirect-param')] = getValue(jq);
      });
      let separator = path.match(/\?/) ? '&' : '?';
      // Webflow will redirect according to this "redirect" attribute.
      data.redirect = path + separator + $.param(params) + fragment;
      originalHandler(data);
    };

    // Upon initial page load, formJq.data('.w-form') doesn't exist yet.
    // So, we wait until submission time to set up the redirection handler.
    formJq.find('input[type=submit]').click(() => {
      let data = formJq.data('.w-form');  // Webflow puts the form data here
      if (!originalHandler) originalHandler = data.handler;
      data.handler = submitAndRedirect;
    });
  });
};

// Sets up autocompletion for fields with the x-autocompletions attribute.
setupAutocompletions = () => {
  eachJq($('[x-autocompletions]'), (fieldJq) => {
    fieldJq.autocomplete({
      delay: 0,
      minLength: 0,
      source: eval(fieldJq.attr('x-autocompletions'))
    });
    // When the field gains focus, pop open the menu.
    fieldJq.focus(() => fieldJq.autocomplete('search', fieldJq.val()));
    if (fieldJq.attr('x-onautocompleteselect')) {
      fieldJq.on('autocompleteselect', () => eval(
        fieldJq.attr('x-onautocompleteselect')
      ));
    }
  });
};

// Sets up a validator function that (a) continuously updates a warning
// indicator while the user is typing or pasting into the field and (b)
// normalizes the contents of the field when the focus leaves the field.
setValidator = (inputJq, validator) => {
  inputJq.on('input propertychange', event => {
    const present = event.target.value.trim();
    const invalid = validator(event.target.value) == null;
    inputJq.attr('x-invalid', present && invalid ? 1 : 0);
  }).on('change', event => {
    const normalizedValue = validator(event.target.value);
    if (normalizedValue != null) event.target.value = normalizedValue;
  });
};

// Sets up an input filter function to filter the contents of the input field
// continuously as the user is typing or pasting into it.  Use sparingly.
setInputFilter = (inputJq, inputFilter) => {
  inputJq.on('input propertychange', event => {
    event.target.value = inputFilter(event.target.value);
  });
};

// Sets up validation logic according to x-validator and x-input-filter.
setupFormValidation = () => {
  eachJq($('[required]'), inputJq =>
    inputJq.on('input propertychange change', event =>
      inputJq.attr('x-missing', event.target.value.trim() ? 0 : 1)));
  eachJq($('[x-validator]'), inputJq =>
    setValidator(inputJq, evalJs(inputJq, inputJq.attr('x-validator'))));
  eachJq($('[x-input-filter]'), inputJq =>
    setInputFilter(inputJq, evalJs(inputJq, inputJq.attr('x-input-filter'))));
};

setupMultistepForm = (selector) => {
  const rootJq = $(selector);
  if (rootJq.length) {
    $.getScript('https://cdn.jsdelivr.net/gh/brotame/advanced-webflow-forms@1/dist/awf.js').done(() => {
      new AWF.MSF({
        formSelector: selector,
        backSelector: '[x-id=previous-button]',
        nextSelector: '[x-id=next-button]',
        alertText: 'Fields marked with * are required.',
        warningClass: 'x-form-warning'
      });

      // Set up OptionSets for any containers with the x-option-set attribute.
      const optionSets = mapJq(rootJq.find('[x-option-set]'), createOptionSet);
      const backJq = $('[x-id=previous-button]');
      const nextJq = $('[x-id=next-button]');
      const numSteps = $('.step').length;
      const currentStep = () => $('.step').index($('.step:not([aria-hidden])'));

      // On the first page only, disable the Back button.
-     backJq.attr('disabled', true);

      nextJq.click(() => {
        backJq.attr('disabled', false);

        var formJq = rootJq.find('form');
        var url = formJq.attr('x-partial-action');
        if (url) postFormAsJson(formJq[0], url);

        // This gets the 0-based step index AFTER the step number has changed.
        var step = currentStep();

        if (window.dataLayer) {
          // Send a Google Analytics 4 event when the user completes a step.
          window.dataLayer.push({
            'event': 'form_page_' + step, 'form': formJq.attr('id')
          });
        }
        if (step === numSteps - 1) {
          // We've arrived at the last page.
          $('[x-hide-on-last-page=1]').hide();
        } else {
          $('[x-hide-on-last-page=1]').show();
        }
        if (step === numSteps) {
          // The Submit button has been clicked on the last page.
          optionSets.forEach(optionSet => optionSet.finalize());
        }
      });
    });
  }
};

setupAllMultistepForms = () => {
  let lastFormId = 1;
  // Form Blocks (.w-form) with a "Next" button are considered multistep forms.
  // Note that Webflow Form Blocks are not <form> elements, but their parents!
  eachJq($('[x-id=next-button]').closest('.w-form'), formJq => {
    // setupMultistepForm needs a selector, not an element.
    formJq.attr('x-form-id', ++lastFormId);
    setupMultistepForm(`[x-form-id="${lastFormId}"]`);
  });
};

setupDatePickers = () => {
  let pickersJq = $('input[x-date-picker=1]');
  if (pickersJq.length) {
    if ($.datepicker) pickersJq.datepicker();
    else {
      // jquery-ui provides a nice date picker widget.
      $('head').append('<link rel="stylesheet" href="https://code.jquery.com/ui/1.12.1/themes/base/jquery-ui.css">');
      $.getScript(
        'https://code.jquery.com/ui/1.12.1/jquery-ui.js'
      ).done(setupDatePickers);
    }
  }
};

setupDismissables = () => {
  let dismissablesJq = $('[x-dismissable=1]');
  if (dismissablesJq.length) {
    // js-cookie provides the Cookies object.
    $.getScript(
      'https://static.techforcampaigns.org/js-cookie-3.0.1.min.js'
    ).done(() => eachNode(dismissablesJq, dis =>
      $(dis).attr('x-dismissed', Cookies.get('dismissed.' + dis.id) ? 1 : 0)
    ));
  }
};

setupReservedAttributes = () => {
  // Webflow won't let us set onclick, onfocus, oninput, or type.  To customize
  // these attributes, we set x-onclick, x-onfocus, x-oninput, or x-type in
  // Webflow and then use this bit of JavaScript to copy over their values.
  eachJq($('[x-onclick]'), jq => jq.attr('onclick', jq.attr('x-onclick')));
  eachJq($('[x-onfocus]'), jq => jq.attr('onfocus', jq.attr('x-onfocus')));
  eachJq($('[x-oninput]'), jq => jq.attr('oninput', jq.attr('x-oninput')));
  eachJq($('[x-type]'), jq => jq.attr('type', jq.attr('x-type')));
};

setupFormPrefill = () => {
  if ($.deparam) {
    let params = $.deparam.querystring();
    for (let key of Object.keys(params)) {
      $('input#' + key).val(params[key]);
    }
  } else {
    // jQuery BBQ provides $.deparam().
    $.browser = {};  // for compatibility with an older version of jQuery
    $.getScript(
      'https://static.techforcampaigns.org/jquery-bbq-1.2.1.min.js'
    ).done(setupFormPrefill);
  }
};

// Gets the Google ID Token either from the given CredentialResponse (when
// invoked as a callback by the Sign In With Google API), or from local storage.
getJwt = (response) => {
  let jwt = response?.credential;
  if (jwt) localStorage.setItem('google_jwt', jwt);
  return localStorage.getItem('google_jwt');
};

// Decodes a Google ID Token into a payload object, unless the token has
// expired.  The contents of a Google ID Token payload are documented here:
// https://developers.google.com/identity/protocols/oauth2/openid-connect#an-id-tokens-payload
decodeJwt = (jwt) => {
  let payload = jwt ? JSON.parse(atob(jwt.split('.')[1])) : {};
  let issuedAt = payload['iat'];
  let nowSeconds = new Date().getTime()/1000;
  if (nowSeconds < issuedAt + APPARENT_JWT_TTL) return payload;
};

// Builds our sign-in UI inside the given container element, which consists of:
// (a) A generic Sign In With Google button, which the user clicks to sign in
// (b) A personal profile icon, which is shown if the user is already signed in
// At any given moment, exactly one of (a) or (b) is visible.
buildGoogleIconUi = (rootJq) => {
  rootJq.append($('<div x-id="generic-icon" x-hidden-if-authed="1">'));
  rootJq.append($('<div x-id="personal-icon" x-shown-if-authed="1">'));
};

buildGoogleButtonUi = (rootJq) => {
  rootJq.append($('<div x-id="generic-google-button" x-hidden-if-authed="1">'));
};

// Updates the sign-in UI to reflect the user's sign-in status.  This can be
// invoked as a callback from the Sign In With Google API when sign-in takes
// place, or called directly to make the UI reflect the current status.
updateSignInUi = (response) => {
  let creds = decodeJwt(getJwt(response));

  // If we're being called back with a signed-in token from the Sign In With
  // Google API, record a sign-in event with GA4.
  if (response && creds && window.dataLayer) {
    window.dataLayer.push({'event': 'sign_in', 'email': creds?.email});
  }
  flags.authed = creds;
  $('[x-id=personal-icon]').css('background-image', `url('${creds?.picture}')`);
  $('[x-id=google-account-link]').text(creds?.email);
};

// Clears the credentials in local storage and in the Sign In With Google API.
signOut = () => {
  let creds = decodeJwt(getJwt());
  if (window.dataLayer) {
    window.dataLayer.push({'event': 'sign_out', 'email': creds?.email});
  }
  localStorage.removeItem('google_jwt');
  google.accounts.id.revoke(creds?.sub);
  updateSignInUi();
};

// Sets up Sign In With Google on the current page.
// You can sign in or out in either place: the icon or the button.
setupSignInUi = () => {
  let iconContainerJq = $('[x-id=google-sign-in]');
  let buttonContainerJq = $('[x-id=google-button-sign-in]');
  if (iconContainerJq.length || buttonContainerJq.length) {
    // If the user is signed in, we can show the profile icon immediately.
    if (iconContainerJq.length) buildGoogleIconUi(iconContainerJq);
    if (buttonContainerJq.length) buildGoogleButtonUi(buttonContainerJq);
    $('[x-id=sign-out-link]').click(signOut);
    updateSignInUi();

    // Then we load the Sign In With Google API and connect it to the UI.
    $.getScript('https://accounts.google.com/gsi/client').done(() => {
      google.accounts.id.initialize({
        client_id: '851439851047-l07ra16vgj7f17boef6oodc7pd8guqa9.apps.googleusercontent.com',
        callback: updateSignInUi
      });
      eachNode($('[x-id=generic-icon]'), node =>
        google.accounts.id.renderButton(
          node, {type: 'icon', shape: 'circle', theme: 'outline', size: 'medium'}
        )
      );
      eachNode($('[x-id=generic-google-button]'), node =>
        google.accounts.id.renderButton(
          node, {type: 'standard', theme: 'outline', size: 'large'}
        )
      );
    });
  }
};

// Replaces the label on an element with the given text for a couple seconds.
function brieflyChangeText(element, text, timeout) {
  let originalText = $(element).text();
  let originalWidth = element.offsetWidth;
  $(element).text(text);
  element.style.minWidth = originalWidth + 'px';  // avoid distracting resizing
  window.setTimeout(() => $(element).text(originalText), timeout || 2000);
}

// Copies some text to the clipboard, updating the clicked button with the
// message "Copied!" for a couple seconds.
function copyToClipboard(text, button) {
  navigator.clipboard.writeText(text);
  if (button) brieflyChangeText(button, 'Copied!');
}

// Indents list items that begin with ">".  This is a workaround for the fact
// that Webflow doesn't support nested lists in Rich Text fields.  See tfc.css
// for the styles applied according to the "level" attribute.
function indentListItems() {
  $('li').each(function() {
    text = $(this).text();
    m = text.match(/^([ >]*)(.*)/);
    levels = (m[1].match(/>/g) || []).length;
    if (levels) $(this).text(m[2]).attr('level', levels);
  });
}

// These are all steps that take place before the template is rendered.
// We don't yet handle forms that are generated by looping templates, though
// forms can contain input elements generated by looping templates.
indentListItems();
setupAllMultistepForms();
setupAutocompletions();
setupDismissables();
setupDomainFields();
setupFormRedirection();
setupFormValidation();
setupReservedAttributes();
setupSignInUi();
setupNavbarToggler();
setupRfvForm();

// setupAllRecaptchaForms() must be called last, because we want it to have
// first priority to intercept form submissions.
setupAllRecaptchaForms();

handleRenderComplete = () => {
  gatherHeadElements();
  setupDatePickers();
  setupFormPrefill();
  revealByFragment();
  showHideByFragment();
  flags.ready = 1;
}

renderAllTemplates();
