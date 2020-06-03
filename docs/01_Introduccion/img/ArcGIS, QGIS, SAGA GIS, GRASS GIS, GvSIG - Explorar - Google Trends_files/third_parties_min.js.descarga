//javascript/closure/base.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Bootstrap for the Google JS Library (Closure).
 *
 * In uncompiled mode base.js will attempt to load Closure's deps file, unless
 * the global <code>CLOSURE_NO_DEPS</code> is set to true.  This allows projects
 * to include their own deps file(s) from different locations.
 *
 * Avoid including base.js more than once. This is strictly discouraged and not
 * supported. goog.require(...) won't work properly in that case.
 *
 * @provideGoog
 */


/**
 * @define {boolean} Overridden to true by the compiler.
 */
var COMPILED = false;


/**
 * Base namespace for the Closure library.  Checks to see goog is already
 * defined in the current scope before assigning to prevent clobbering if
 * base.js is loaded more than once.
 *
 * @const
 */
var goog = goog || {};

/**
 * Reference to the global object.
 * https://www.ecma-international.org/ecma-262/9.0/index.html#sec-global-object
 *
 * More info on this implementation here:
 * https://docs.google.com/document/d/1NAeW4Wk7I7FV0Y2tcUFvQdGMc89k2vdgSXInw8_nvCI/edit
 *
 * @const
 * @suppress {undefinedVars} self won't be referenced unless `this` is falsy.
 * @type {!Global}
 */
goog.global =
    // Check `this` first for backwards compatibility.
    // Valid unless running as an ES module or in a function wrapper called
    //   without setting `this` properly.
    // Note that base.js can't usefully be imported as an ES module, but it may
    // be compiled into bundles that are loadable as ES modules.
    this ||
    // https://developer.mozilla.org/en-US/docs/Web/API/Window/self
    // For in-page browser environments and workers.
    self;


/**
 * A hook for overriding the define values in uncompiled mode.
 *
 * In uncompiled mode, `CLOSURE_UNCOMPILED_DEFINES` may be defined before
 * loading base.js.  If a key is defined in `CLOSURE_UNCOMPILED_DEFINES`,
 * `goog.define` will use the value instead of the default value.  This
 * allows flags to be overwritten without compilation (this is normally
 * accomplished with the compiler's "define" flag).
 *
 * Example:
 * <pre>
 *   var CLOSURE_UNCOMPILED_DEFINES = {'goog.DEBUG': false};
 * </pre>
 *
 * @type {Object<string, (string|number|boolean)>|undefined}
 */
goog.global.CLOSURE_UNCOMPILED_DEFINES;


/**
 * A hook for overriding the define values in uncompiled or compiled mode,
 * like CLOSURE_UNCOMPILED_DEFINES but effective in compiled code.  In
 * uncompiled code CLOSURE_UNCOMPILED_DEFINES takes precedence.
 *
 * Also unlike CLOSURE_UNCOMPILED_DEFINES the values must be number, boolean or
 * string literals or the compiler will emit an error.
 *
 * While any @define value may be set, only those set with goog.define will be
 * effective for uncompiled code.
 *
 * Example:
 * <pre>
 *   var CLOSURE_DEFINES = {'goog.DEBUG': false} ;
 * </pre>
 *
 * @type {Object<string, (string|number|boolean)>|undefined}
 */
goog.global.CLOSURE_DEFINES;


/**
 * Builds an object structure for the provided namespace path, ensuring that
 * names that already exist are not overwritten. For example:
 * "a.b.c" -> a = {};a.b={};a.b.c={};
 * Used by goog.provide and goog.exportSymbol.
 * @param {string} name name of the object that this file defines.
 * @param {*=} opt_object the object to expose at the end of the path.
 * @param {Object=} opt_objectToExportTo The object to add the path to; default
 *     is `goog.global`.
 * @private
 */
goog.exportPath_ = function(name, opt_object, opt_objectToExportTo) {
  var parts = name.split('.');
  var cur = opt_objectToExportTo || goog.global;

  // Internet Explorer exhibits strange behavior when throwing errors from
  // methods externed in this manner.  See the testExportSymbolExceptions in
  // base_test.html for an example.
  if (!(parts[0] in cur) && typeof cur.execScript != 'undefined') {
    cur.execScript('var ' + parts[0]);
  }

  for (var part; parts.length && (part = parts.shift());) {
    if (!parts.length && opt_object !== undefined) {
      // last part and we have an object; use it
      cur[part] = opt_object;
    } else if (cur[part] && cur[part] !== Object.prototype[part]) {
      cur = cur[part];
    } else {
      cur = cur[part] = {};
    }
  }
};


/**
 * Defines a named value. In uncompiled mode, the value is retrieved from
 * CLOSURE_DEFINES or CLOSURE_UNCOMPILED_DEFINES if the object is defined and
 * has the property specified, and otherwise used the defined defaultValue.
 * When compiled the default can be overridden using the compiler options or the
 * value set in the CLOSURE_DEFINES object. Returns the defined value so that it
 * can be used safely in modules. Note that the value type MUST be either
 * boolean, number, or string.
 *
 * @param {string} name The distinguished name to provide.
 * @param {T} defaultValue
 * @return {T} The defined value.
 * @template T
 */
goog.define = function(name, defaultValue) {
  var value = defaultValue;
  if (!COMPILED) {
    var uncompiledDefines = goog.global.CLOSURE_UNCOMPILED_DEFINES;
    var defines = goog.global.CLOSURE_DEFINES;
    if (uncompiledDefines &&
        // Anti DOM-clobbering runtime check (b/37736576).
        /** @type {?} */ (uncompiledDefines).nodeType === undefined &&
        Object.prototype.hasOwnProperty.call(uncompiledDefines, name)) {
      value = uncompiledDefines[name];
    } else if (
        defines &&
        // Anti DOM-clobbering runtime check (b/37736576).
        /** @type {?} */ (defines).nodeType === undefined &&
        Object.prototype.hasOwnProperty.call(defines, name)) {
      value = defines[name];
    }
  }
  return value;
};


/**
 * @define {number} Integer year indicating the set of browser features that are
 * guaranteed to be present.  This is defined to include exactly features that
 * work correctly on all "modern" browsers that are stable on January 1 of the
 * specified year.  For example,
 * ```js
 * if (goog.FEATURESET_YEAR >= 2019) {
 *   // use APIs known to be available on all major stable browsers Jan 1, 2019
 * } else {
 *   // polyfill for older browsers
 * }
 * ```
 * This is intended to be the primary define for removing
 * unnecessary browser compatibility code (such as ponyfills and workarounds),
 * and should inform the default value for most other defines:
 * ```js
 * const ASSUME_NATIVE_PROMISE =
 *     goog.define('ASSUME_NATIVE_PROMISE', goog.FEATURESET_YEAR >= 2016);
 * ```
 *
 * The default assumption is that IE9 is the lowest supported browser, which was
 * first available Jan 1, 2012.
 *
 * TODO(mathiasb): Reference more thorough documentation when it's available.
 */
goog.FEATURESET_YEAR = goog.define('goog.FEATURESET_YEAR', 2012);


/**
 * @define {boolean} DEBUG is provided as a convenience so that debugging code
 * that should not be included in a production. It can be easily stripped
 * by specifying --define goog.DEBUG=false to the Closure Compiler aka
 * JSCompiler. For example, most toString() methods should be declared inside an
 * "if (goog.DEBUG)" conditional because they are generally used for debugging
 * purposes and it is difficult for the JSCompiler to statically determine
 * whether they are used.
 */
goog.DEBUG = goog.define('goog.DEBUG', true);


/**
 * @define {string} LOCALE defines the locale being used for compilation. It is
 * used to select locale specific data to be compiled in js binary. BUILD rule
 * can specify this value by "--define goog.LOCALE=<locale_name>" as a compiler
 * option.
 *
 * Take into account that the locale code format is important. You should use
 * the canonical Unicode format with hyphen as a delimiter. Language must be
 * lowercase, Language Script - Capitalized, Region - UPPERCASE.
 * There are few examples: pt-BR, en, en-US, sr-Latin-BO, zh-Hans-CN.
 *
 * See more info about locale codes here:
 * http://www.unicode.org/reports/tr35/#Unicode_Language_and_Locale_Identifiers
 *
 * For language codes you should use values defined by ISO 693-1. See it here
 * http://www.w3.org/WAI/ER/IG/ert/iso639.htm. There is only one exception from
 * this rule: the Hebrew language. For legacy reasons the old code (iw) should
 * be used instead of the new code (he).
 *
 * MOE:begin_intracomment_strip
 * See http://g3doc/i18n/identifiers/g3doc/synonyms.
 * MOE:end_intracomment_strip
 */
goog.LOCALE = goog.define('goog.LOCALE', 'en');  // default to en


/**
 * @define {boolean} Whether this code is running on trusted sites.
 *
 * On untrusted sites, several native functions can be defined or overridden by
 * external libraries like Prototype, Datejs, and JQuery and setting this flag
 * to false forces closure to use its own implementations when possible.
 *
 * If your JavaScript can be loaded by a third party site and you are wary about
 * relying on non-standard implementations, specify
 * "--define goog.TRUSTED_SITE=false" to the compiler.
 */
goog.TRUSTED_SITE = goog.define('goog.TRUSTED_SITE', true);


/**
 * @define {boolean} Whether a project is expected to be running in strict mode.
 *
 * This define can be used to trigger alternate implementations compatible with
 * running in EcmaScript Strict mode or warn about unavailable functionality.
 * @see https://goo.gl/PudQ4y
 *
 */
goog.STRICT_MODE_COMPATIBLE = goog.define('goog.STRICT_MODE_COMPATIBLE', false);


/**
 * @define {boolean} Whether code that calls {@link goog.setTestOnly} should
 *     be disallowed in the compilation unit.
 */
goog.DISALLOW_TEST_ONLY_CODE =
    goog.define('goog.DISALLOW_TEST_ONLY_CODE', COMPILED && !goog.DEBUG);


/**
 * @define {boolean} Whether to use a Chrome app CSP-compliant method for
 *     loading scripts via goog.require. @see appendScriptSrcNode_.
 */
goog.ENABLE_CHROME_APP_SAFE_SCRIPT_LOADING =
    goog.define('goog.ENABLE_CHROME_APP_SAFE_SCRIPT_LOADING', false);


/**
 * Defines a namespace in Closure.
 *
 * A namespace may only be defined once in a codebase. It may be defined using
 * goog.provide() or goog.module().
 *
 * The presence of one or more goog.provide() calls in a file indicates
 * that the file defines the given objects/namespaces.
 * Provided symbols must not be null or undefined.
 *
 * In addition, goog.provide() creates the object stubs for a namespace
 * (for example, goog.provide("goog.foo.bar") will create the object
 * goog.foo.bar if it does not already exist).
 *
 * Build tools also scan for provide/require/module statements
 * to discern dependencies, build dependency files (see deps.js), etc.
 *
 * @see goog.require
 * @see goog.module
 * @param {string} name Namespace provided by this file in the form
 *     "goog.package.part".
 */
goog.provide = function(name) {
  if (goog.isInModuleLoader_()) {
    throw new Error('goog.provide cannot be used within a module.');
  }
  if (!COMPILED) {
    // Ensure that the same namespace isn't provided twice.
    // A goog.module/goog.provide maps a goog.require to a specific file
    if (goog.isProvided_(name)) {
      throw new Error('Namespace "' + name + '" already declared.');
    }
  }

  goog.constructNamespace_(name);
};


/**
 * @param {string} name Namespace provided by this file in the form
 *     "goog.package.part".
 * @param {Object=} opt_obj The object to embed in the namespace.
 * @private
 */
goog.constructNamespace_ = function(name, opt_obj) {
  if (!COMPILED) {
    delete goog.implicitNamespaces_[name];

    var namespace = name;
    while ((namespace = namespace.substring(0, namespace.lastIndexOf('.')))) {
      if (goog.getObjectByName(namespace)) {
        break;
      }
      goog.implicitNamespaces_[namespace] = true;
    }
  }

  goog.exportPath_(name, opt_obj);
};


/**
 * Returns CSP nonce, if set for any script tag.
 * @param {?Window=} opt_window The window context used to retrieve the nonce.
 *     Defaults to global context.
 * @return {string} CSP nonce or empty string if no nonce is present.
 */
goog.getScriptNonce = function(opt_window) {
  if (opt_window && opt_window != goog.global) {
    return goog.getScriptNonce_(opt_window.document);
  }
  if (goog.cspNonce_ === null) {
    goog.cspNonce_ = goog.getScriptNonce_(goog.global.document);
  }
  return goog.cspNonce_;
};


/**
 * According to the CSP3 spec a nonce must be a valid base64 string.
 * @see https://www.w3.org/TR/CSP3/#grammardef-base64-value
 * @private @const
 */
goog.NONCE_PATTERN_ = /^[\w+/_-]+[=]{0,2}$/;


/**
 * @private {?string}
 */
goog.cspNonce_ = null;


/**
 * Returns CSP nonce, if set for any script tag.
 * @param {!Document} doc
 * @return {string} CSP nonce or empty string if no nonce is present.
 * @private
 */
goog.getScriptNonce_ = function(doc) {
  var script = doc.querySelector && doc.querySelector('script[nonce]');
  if (script) {
    // Try to get the nonce from the IDL property first, because browsers that
    // implement additional nonce protection features (currently only Chrome) to
    // prevent nonce stealing via CSS do not expose the nonce via attributes.
    // See https://github.com/whatwg/html/issues/2369
    var nonce = script['nonce'] || script.getAttribute('nonce');
    if (nonce && goog.NONCE_PATTERN_.test(nonce)) {
      return nonce;
    }
  }
  return '';
};


/**
 * Module identifier validation regexp.
 * Note: This is a conservative check, it is very possible to be more lenient,
 *   the primary exclusion here is "/" and "\" and a leading ".", these
 *   restrictions are intended to leave the door open for using goog.require
 *   with relative file paths rather than module identifiers.
 * @private
 */
goog.VALID_MODULE_RE_ = /^[a-zA-Z_$][a-zA-Z0-9._$]*$/;


/**
 * Defines a module in Closure.
 *
 * Marks that this file must be loaded as a module and claims the namespace.
 *
 * A namespace may only be defined once in a codebase. It may be defined using
 * goog.provide() or goog.module().
 *
 * goog.module() has three requirements:
 * - goog.module may not be used in the same file as goog.provide.
 * - goog.module must be the first statement in the file.
 * - only one goog.module is allowed per file.
 *
 * When a goog.module annotated file is loaded, it is enclosed in
 * a strict function closure. This means that:
 * - any variables declared in a goog.module file are private to the file
 * (not global), though the compiler is expected to inline the module.
 * - The code must obey all the rules of "strict" JavaScript.
 * - the file will be marked as "use strict"
 *
 * NOTE: unlike goog.provide, goog.module does not declare any symbols by
 * itself. If declared symbols are desired, use
 * goog.module.declareLegacyNamespace().
 *
 * MOE:begin_intracomment_strip
 * See the goog.module announcement at http://go/goog.module-announce
 * MOE:end_intracomment_strip
 *
 * See the public goog.module proposal: http://goo.gl/Va1hin
 *
 * @param {string} name Namespace provided by this file in the form
 *     "goog.package.part", is expected but not required.
 * @return {void}
 */
goog.module = function(name) {
  if (typeof name !== 'string' || !name ||
      name.search(goog.VALID_MODULE_RE_) == -1) {
    throw new Error('Invalid module identifier');
  }
  if (!goog.isInGoogModuleLoader_()) {
    throw new Error(
        'Module ' + name + ' has been loaded incorrectly. Note, ' +
        'modules cannot be loaded as normal scripts. They require some kind of ' +
        'pre-processing step. You\'re likely trying to load a module via a ' +
        'script tag or as a part of a concatenated bundle without rewriting the ' +
        'module. For more info see: ' +
        'https://github.com/google/closure-library/wiki/goog.module:-an-ES6-module-like-alternative-to-goog.provide.');
  }
  if (goog.moduleLoaderState_.moduleName) {
    throw new Error('goog.module may only be called once per module.');
  }

  // Store the module name for the loader.
  goog.moduleLoaderState_.moduleName = name;
  if (!COMPILED) {
    // Ensure that the same namespace isn't provided twice.
    // A goog.module/goog.provide maps a goog.require to a specific file
    if (goog.isProvided_(name)) {
      throw new Error('Namespace "' + name + '" already declared.');
    }
    delete goog.implicitNamespaces_[name];
  }
};


/**
 * @param {string} name The module identifier.
 * @return {?} The module exports for an already loaded module or null.
 *
 * Note: This is not an alternative to goog.require, it does not
 * indicate a hard dependency, instead it is used to indicate
 * an optional dependency or to access the exports of a module
 * that has already been loaded.
 * @suppress {missingProvide}
 */
goog.module.get = function(name) {
  return goog.module.getInternal_(name);
};


/**
 * @param {string} name The module identifier.
 * @return {?} The module exports for an already loaded module or null.
 * @private
 */
goog.module.getInternal_ = function(name) {
  if (!COMPILED) {
    if (name in goog.loadedModules_) {
      return goog.loadedModules_[name].exports;
    } else if (!goog.implicitNamespaces_[name]) {
      var ns = goog.getObjectByName(name);
      return ns != null ? ns : null;
    }
  }
  return null;
};


/**
 * Types of modules the debug loader can load.
 * @enum {string}
 */
goog.ModuleType = {
  ES6: 'es6',
  GOOG: 'goog'
};


/**
 * @private {?{
 *   moduleName: (string|undefined),
 *   declareLegacyNamespace:boolean,
 *   type: ?goog.ModuleType
 * }}
 */
goog.moduleLoaderState_ = null;


/**
 * @private
 * @return {boolean} Whether a goog.module or an es6 module is currently being
 *     initialized.
 */
goog.isInModuleLoader_ = function() {
  return goog.isInGoogModuleLoader_() || goog.isInEs6ModuleLoader_();
};


/**
 * @private
 * @return {boolean} Whether a goog.module is currently being initialized.
 */
goog.isInGoogModuleLoader_ = function() {
  return !!goog.moduleLoaderState_ &&
      goog.moduleLoaderState_.type == goog.ModuleType.GOOG;
};


/**
 * @private
 * @return {boolean} Whether an es6 module is currently being initialized.
 */
goog.isInEs6ModuleLoader_ = function() {
  var inLoader = !!goog.moduleLoaderState_ &&
      goog.moduleLoaderState_.type == goog.ModuleType.ES6;

  if (inLoader) {
    return true;
  }

  var jscomp = goog.global['$jscomp'];

  if (jscomp) {
    // jscomp may not have getCurrentModulePath if this is a compiled bundle
    // that has some of the runtime, but not all of it. This can happen if
    // optimizations are turned on so the unused runtime is removed but renaming
    // and Closure pass are off (so $jscomp is still named $jscomp and the
    // goog.provide/require calls still exist).
    if (typeof jscomp.getCurrentModulePath != 'function') {
      return false;
    }

    // Bundled ES6 module.
    return !!jscomp.getCurrentModulePath();
  }

  return false;
};


/**
 * Provide the module's exports as a globally accessible object under the
 * module's declared name.  This is intended to ease migration to goog.module
 * for files that have existing usages.
 * @suppress {missingProvide}
 */
goog.module.declareLegacyNamespace = function() {
  if (!COMPILED && !goog.isInGoogModuleLoader_()) {
    throw new Error(
        'goog.module.declareLegacyNamespace must be called from ' +
        'within a goog.module');
  }
  if (!COMPILED && !goog.moduleLoaderState_.moduleName) {
    throw new Error(
        'goog.module must be called prior to ' +
        'goog.module.declareLegacyNamespace.');
  }
  goog.moduleLoaderState_.declareLegacyNamespace = true;
};


/**
 * Associates an ES6 module with a Closure module ID so that is available via
 * goog.require. The associated ID  acts like a goog.module ID - it does not
 * create any global names, it is merely available via goog.require /
 * goog.module.get / goog.forwardDeclare / goog.requireType. goog.require and
 * goog.module.get will return the entire module as if it was import *'d. This
 * allows Closure files to reference ES6 modules for the sake of migration.
 *
 * @param {string} namespace
 * @suppress {missingProvide}
 */
goog.declareModuleId = function(namespace) {
  if (!COMPILED) {
    if (!goog.isInEs6ModuleLoader_()) {
      throw new Error(
          'goog.declareModuleId may only be called from ' +
          'within an ES6 module');
    }
    if (goog.moduleLoaderState_ && goog.moduleLoaderState_.moduleName) {
      throw new Error(
          'goog.declareModuleId may only be called once per module.');
    }
    if (namespace in goog.loadedModules_) {
      throw new Error(
          'Module with namespace "' + namespace + '" already exists.');
    }
  }
  if (goog.moduleLoaderState_) {
    // Not bundled - debug loading.
    goog.moduleLoaderState_.moduleName = namespace;
  } else {
    // Bundled - not debug loading, no module loader state.
    var jscomp = goog.global['$jscomp'];
    if (!jscomp || typeof jscomp.getCurrentModulePath != 'function') {
      throw new Error(
          'Module with namespace "' + namespace +
          '" has been loaded incorrectly.');
    }
    var exports = jscomp.require(jscomp.getCurrentModulePath());
    goog.loadedModules_[namespace] = {
      exports: exports,
      type: goog.ModuleType.ES6,
      moduleId: namespace
    };
  }
};


/**
 * Marks that the current file should only be used for testing, and never for
 * live code in production.
 *
 * In the case of unit tests, the message may optionally be an exact namespace
 * for the test (e.g. 'goog.stringTest'). The linter will then ignore the extra
 * provide (if not explicitly defined in the code).
 *
 * @param {string=} opt_message Optional message to add to the error that's
 *     raised when used in production code.
 */
goog.setTestOnly = function(opt_message) {
  if (goog.DISALLOW_TEST_ONLY_CODE) {
    opt_message = opt_message || '';
    throw new Error(
        'Importing test-only code into non-debug environment' +
        (opt_message ? ': ' + opt_message : '.'));
  }
};


/**
 * Forward declares a symbol. This is an indication to the compiler that the
 * symbol may be used in the source yet is not required and may not be provided
 * in compilation.
 *
 * The most common usage of forward declaration is code that takes a type as a
 * function parameter but does not need to require it. By forward declaring
 * instead of requiring, no hard dependency is made, and (if not required
 * elsewhere) the namespace may never be required and thus, not be pulled
 * into the JavaScript binary. If it is required elsewhere, it will be type
 * checked as normal.
 *
 * Before using goog.forwardDeclare, please read the documentation at
 * https://github.com/google/closure-compiler/wiki/Bad-Type-Annotation to
 * understand the options and tradeoffs when working with forward declarations.
 *
 * @param {string} name The namespace to forward declare in the form of
 *     "goog.package.part".
 * @deprecated See go/noforwarddeclaration, Use `goog.requireType` instead.
 */
goog.forwardDeclare = function(name) {};


/**
 * Forward declare type information. Used to assign types to goog.global
 * referenced object that would otherwise result in unknown type references
 * and thus block property disambiguation.
 */
goog.forwardDeclare('Document');
goog.forwardDeclare('HTMLScriptElement');
goog.forwardDeclare('XMLHttpRequest');


if (!COMPILED) {
  /**
   * Check if the given name has been goog.provided. This will return false for
   * names that are available only as implicit namespaces.
   * @param {string} name name of the object to look for.
   * @return {boolean} Whether the name has been provided.
   * @private
   */
  goog.isProvided_ = function(name) {
    return (name in goog.loadedModules_) ||
        (!goog.implicitNamespaces_[name] && goog.getObjectByName(name) != null);
  };

  /**
   * Namespaces implicitly defined by goog.provide. For example,
   * goog.provide('goog.events.Event') implicitly declares that 'goog' and
   * 'goog.events' must be namespaces.
   *
   * @type {!Object<string, (boolean|undefined)>}
   * @private
   */
  goog.implicitNamespaces_ = {'goog.module': true};

  // NOTE: We add goog.module as an implicit namespace as goog.module is defined
  // here and because the existing module package has not been moved yet out of
  // the goog.module namespace. This satisifies both the debug loader and
  // ahead-of-time dependency management.
}


/**
 * Returns an object based on its fully qualified external name.  The object
 * is not found if null or undefined.  If you are using a compilation pass that
 * renames property names beware that using this function will not find renamed
 * properties.
 *
 * @param {string} name The fully qualified name.
 * @param {Object=} opt_obj The object within which to look; default is
 *     |goog.global|.
 * @return {?} The value (object or primitive) or, if not found, null.
 */
goog.getObjectByName = function(name, opt_obj) {
  var parts = name.split('.');
  var cur = opt_obj || goog.global;
  for (var i = 0; i < parts.length; i++) {
    cur = cur[parts[i]];
    if (cur == null) {
      return null;
    }
  }
  return cur;
};


/**
 * Globalizes a whole namespace, such as goog or goog.lang.
 *
 * @param {!Object} obj The namespace to globalize.
 * @param {Object=} opt_global The object to add the properties to.
 * @deprecated Properties may be explicitly exported to the global scope, but
 *     this should no longer be done in bulk.
 */
goog.globalize = function(obj, opt_global) {
  var global = opt_global || goog.global;
  for (var x in obj) {
    global[x] = obj[x];
  }
};


/**
 * Adds a dependency from a file to the files it requires.
 * @param {string} relPath The path to the js file.
 * @param {!Array<string>} provides An array of strings with
 *     the names of the objects this file provides.
 * @param {!Array<string>} requires An array of strings with
 *     the names of the objects this file requires.
 * @param {boolean|!Object<string>=} opt_loadFlags Parameters indicating
 *     how the file must be loaded.  The boolean 'true' is equivalent
 *     to {'module': 'goog'} for backwards-compatibility.  Valid properties
 *     and values include {'module': 'goog'} and {'lang': 'es6'}.
 */
goog.addDependency = function(relPath, provides, requires, opt_loadFlags) {
  if (!COMPILED && goog.DEPENDENCIES_ENABLED) {
    goog.debugLoader_.addDependency(relPath, provides, requires, opt_loadFlags);
  }
};


// MOE:begin_strip
/**
 * Whether goog.require should throw an exception if it fails.
 * @type {boolean}
 */
goog.useStrictRequires = false;


// MOE:end_strip


// NOTE(nnaze): The debug DOM loader was included in base.js as an original way
// to do "debug-mode" development.  The dependency system can sometimes be
// confusing, as can the debug DOM loader's asynchronous nature.
//
// With the DOM loader, a call to goog.require() is not blocking -- the script
// will not load until some point after the current script.  If a namespace is
// needed at runtime, it needs to be defined in a previous script, or loaded via
// require() with its registered dependencies.
//
// User-defined namespaces may need their own deps file. For a reference on
// creating a deps file, see:
// MOE:begin_strip
// Internally: http://go/deps-files and http://go/be#js_deps
// MOE:end_strip
// Externally: https://developers.google.com/closure/library/docs/depswriter
//
// Because of legacy clients, the DOM loader can't be easily removed from
// base.js.  Work was done to make it disableable or replaceable for
// different environments (DOM-less JavaScript interpreters like Rhino or V8,
// for example). See bootstrap/ for more information.


/**
 * @define {boolean} Whether to enable the debug loader.
 *
 * If enabled, a call to goog.require() will attempt to load the namespace by
 * appending a script tag to the DOM (if the namespace has been registered).
 *
 * If disabled, goog.require() will simply assert that the namespace has been
 * provided (and depend on the fact that some outside tool correctly ordered
 * the script).
 */
goog.ENABLE_DEBUG_LOADER = goog.define('goog.ENABLE_DEBUG_LOADER', true);


/**
 * @param {string} msg
 * @private
 */
goog.logToConsole_ = function(msg) {
  if (goog.global.console) {
    goog.global.console['error'](msg);
  }
};


/**
 * Implements a system for the dynamic resolution of dependencies that works in
 * parallel with the BUILD system.
 *
 * Note that all calls to goog.require will be stripped by the compiler.
 *
 * @see goog.provide
 * @param {string} namespace Namespace (as was given in goog.provide,
 *     goog.module, or goog.declareModuleId) in the form
 *     "goog.package.part".
 * @return {?} If called within a goog.module or ES6 module file, the associated
 *     namespace or module otherwise null.
 */
goog.require = function(namespace) {
  if (!COMPILED) {
    // Might need to lazy load on old IE.
    if (goog.ENABLE_DEBUG_LOADER) {
      goog.debugLoader_.requested(namespace);
    }

    // If the object already exists we do not need to do anything.
    if (goog.isProvided_(namespace)) {
      if (goog.isInModuleLoader_()) {
        return goog.module.getInternal_(namespace);
      }
    } else if (goog.ENABLE_DEBUG_LOADER) {
      var moduleLoaderState = goog.moduleLoaderState_;
      goog.moduleLoaderState_ = null;
      try {
        goog.debugLoader_.load_(namespace);
      } finally {
        goog.moduleLoaderState_ = moduleLoaderState;
      }
    }

    return null;
  }
};


/**
 * Requires a symbol for its type information. This is an indication to the
 * compiler that the symbol may appear in type annotations, yet it is not
 * referenced at runtime.
 *
 * When called within a goog.module or ES6 module file, the return value may be
 * assigned to or destructured into a variable, but it may not be otherwise used
 * in code outside of a type annotation.
 *
 * Note that all calls to goog.requireType will be stripped by the compiler.
 *
 * @param {string} namespace Namespace (as was given in goog.provide,
 *     goog.module, or goog.declareModuleId) in the form
 *     "goog.package.part".
 * @return {?}
 */
goog.requireType = function(namespace) {
  // Return an empty object so that single-level destructuring of the return
  // value doesn't crash at runtime when using the debug loader. Multi-level
  // destructuring isn't supported.
  return {};
};


/**
 * Path for included scripts.
 * @type {string}
 */
goog.basePath = '';


/**
 * A hook for overriding the base path.
 * @type {string|undefined}
 */
goog.global.CLOSURE_BASE_PATH;


/**
 * Whether to attempt to load Closure's deps file. By default, when uncompiled,
 * deps files will attempt to be loaded.
 * @type {boolean|undefined}
 */
goog.global.CLOSURE_NO_DEPS;


/**
 * A function to import a single script. This is meant to be overridden when
 * Closure is being run in non-HTML contexts, such as web workers. It's defined
 * in the global scope so that it can be set before base.js is loaded, which
 * allows deps.js to be imported properly.
 *
 * The first parameter the script source, which is a relative URI. The second,
 * optional parameter is the script contents, in the event the script needed
 * transformation. It should return true if the script was imported, false
 * otherwise.
 * @type {(function(string, string=): boolean)|undefined}
 */
goog.global.CLOSURE_IMPORT_SCRIPT;


/**
 * Null function used for default values of callbacks, etc.
 * @return {void} Nothing.
 */
goog.nullFunction = function() {};


/**
 * When defining a class Foo with an abstract method bar(), you can do:
 * Foo.prototype.bar = goog.abstractMethod
 *
 * Now if a subclass of Foo fails to override bar(), an error will be thrown
 * when bar() is invoked.
 *
 * @type {!Function}
 * @throws {Error} when invoked to indicate the method should be overridden.
 * @deprecated Use "@abstract" annotation instead of goog.abstractMethod in new
 *     code. See
 *     https://github.com/google/closure-compiler/wiki/@abstract-classes-and-methods
 */
goog.abstractMethod = function() {
  throw new Error('unimplemented abstract method');
};


/**
 * Adds a `getInstance` static method that always returns the same
 * instance object.
 * @param {!Function} ctor The constructor for the class to add the static
 *     method to.
 * @suppress {missingProperties} 'instance_' isn't a property on 'Function'
 *     but we don't have a better type to use here.
 */
goog.addSingletonGetter = function(ctor) {
  // instance_ is immediately set to prevent issues with sealed constructors
  // such as are encountered when a constructor is returned as the export object
  // of a goog.module in unoptimized code.
  // Delcare type to avoid conformance violations that ctor.instance_ is unknown
  /** @type {undefined|!Object} @suppress {underscore} */
  ctor.instance_ = undefined;
  ctor.getInstance = function() {
    if (ctor.instance_) {
      return ctor.instance_;
    }
    if (goog.DEBUG) {
      // NOTE: JSCompiler can't optimize away Array#push.
      goog.instantiatedSingletons_[goog.instantiatedSingletons_.length] = ctor;
    }
    // Cast to avoid conformance violations that ctor.instance_ is unknown
    return /** @type {!Object|undefined} */ (ctor.instance_) = new ctor;
  };
};


/**
 * All singleton classes that have been instantiated, for testing. Don't read
 * it directly, use the `goog.testing.singleton` module. The compiler
 * removes this variable if unused.
 * @type {!Array<!Function>}
 * @private
 */
goog.instantiatedSingletons_ = [];


/**
 * @define {boolean} Whether to load goog.modules using `eval` when using
 * the debug loader.  This provides a better debugging experience as the
 * source is unmodified and can be edited using Chrome Workspaces or similar.
 * However in some environments the use of `eval` is banned
 * so we provide an alternative.
 */
goog.LOAD_MODULE_USING_EVAL = goog.define('goog.LOAD_MODULE_USING_EVAL', true);


/**
 * @define {boolean} Whether the exports of goog.modules should be sealed when
 * possible.
 */
goog.SEAL_MODULE_EXPORTS = goog.define('goog.SEAL_MODULE_EXPORTS', goog.DEBUG);


/**
 * The registry of initialized modules:
 * The module identifier or path to module exports map.
 * @private @const {!Object<string, {exports:?,type:string,moduleId:string}>}
 */
goog.loadedModules_ = {};


/**
 * True if the debug loader enabled and used.
 * @const {boolean}
 */
goog.DEPENDENCIES_ENABLED = !COMPILED && goog.ENABLE_DEBUG_LOADER;


/**
 * @define {string} How to decide whether to transpile.  Valid values
 * are 'always', 'never', and 'detect'.  The default ('detect') is to
 * use feature detection to determine which language levels need
 * transpilation.
 */
// NOTE(sdh): we could expand this to accept a language level to bypass
// detection: e.g. goog.TRANSPILE == 'es5' would transpile ES6 files but
// would leave ES3 and ES5 files alone.
goog.TRANSPILE = goog.define('goog.TRANSPILE', 'detect');

/**
 * @define {boolean} If true assume that ES modules have already been
 * transpiled by the jscompiler (in the same way that transpile.js would
 * transpile them - to jscomp modules). Useful only for servers that wish to use
 * the debug loader and transpile server side. Thus this is only respected if
 * goog.TRANSPILE is "never".
 */
goog.ASSUME_ES_MODULES_TRANSPILED =
    goog.define('goog.ASSUME_ES_MODULES_TRANSPILED', false);


/**
 * @define {string} If a file needs to be transpiled what the output language
 * should be. By default this is the highest language level this file detects
 * the current environment supports. Generally this flag should not be set, but
 * it could be useful to override. Example: If the current environment supports
 * ES6 then by default ES7+ files will be transpiled to ES6, unless this is
 * overridden.
 *
 * Valid values include: es3, es5, es6, es7, and es8. Anything not recognized
 * is treated as es3.
 *
 * Note that setting this value does not force transpilation. Just if
 * transpilation occurs this will be the output. So this is most useful when
 * goog.TRANSPILE is set to 'always' and then forcing the language level to be
 * something lower than what the environment detects.
 */
goog.TRANSPILE_TO_LANGUAGE = goog.define('goog.TRANSPILE_TO_LANGUAGE', '');


/**
 * @define {string} Path to the transpiler.  Executing the script at this
 * path (relative to base.js) should define a function $jscomp.transpile.
 */
goog.TRANSPILER = goog.define('goog.TRANSPILER', 'transpile.js');


/**
 * @package {?boolean}
 * Visible for testing.
 */
goog.hasBadLetScoping = null;


/**
 * @return {boolean}
 * @package Visible for testing.
 */
goog.useSafari10Workaround = function() {
  if (goog.hasBadLetScoping == null) {
    var hasBadLetScoping;
    try {
      hasBadLetScoping = !eval(
          '"use strict";' +
          'let x = 1; function f() { return typeof x; };' +
          'f() == "number";');
    } catch (e) {
      // Assume that ES6 syntax isn't supported.
      hasBadLetScoping = false;
    }
    goog.hasBadLetScoping = hasBadLetScoping;
  }
  return goog.hasBadLetScoping;
};


/**
 * @param {string} moduleDef
 * @return {string}
 * @package Visible for testing.
 */
goog.workaroundSafari10EvalBug = function(moduleDef) {
  return '(function(){' + moduleDef +
      '\n' +  // Terminate any trailing single line comment.
      ';' +   // Terminate any trailing expression.
      '})();\n';
};


/**
 * @param {function(?):?|string} moduleDef The module definition.
 */
goog.loadModule = function(moduleDef) {
  // NOTE: we allow function definitions to be either in the from
  // of a string to eval (which keeps the original source intact) or
  // in a eval forbidden environment (CSP) we allow a function definition
  // which in its body must call `goog.module`, and return the exports
  // of the module.
  var previousState = goog.moduleLoaderState_;
  try {
    goog.moduleLoaderState_ = {
      moduleName: '',
      declareLegacyNamespace: false,
      type: goog.ModuleType.GOOG
    };
    var exports;
    if (goog.isFunction(moduleDef)) {
      exports = moduleDef.call(undefined, {});
    } else if (typeof moduleDef === 'string') {
      if (goog.useSafari10Workaround()) {
        moduleDef = goog.workaroundSafari10EvalBug(moduleDef);
      }

      exports = goog.loadModuleFromSource_.call(undefined, moduleDef);
    } else {
      throw new Error('Invalid module definition');
    }

    var moduleName = goog.moduleLoaderState_.moduleName;
    if (typeof moduleName === 'string' && moduleName) {
      // Don't seal legacy namespaces as they may be used as a parent of
      // another namespace
      if (goog.moduleLoaderState_.declareLegacyNamespace) {
        goog.constructNamespace_(moduleName, exports);
      } else if (
          goog.SEAL_MODULE_EXPORTS && Object.seal &&
          typeof exports == 'object' && exports != null) {
        Object.seal(exports);
      }

      var data = {
        exports: exports,
        type: goog.ModuleType.GOOG,
        moduleId: goog.moduleLoaderState_.moduleName
      };
      goog.loadedModules_[moduleName] = data;
    } else {
      throw new Error('Invalid module name \"' + moduleName + '\"');
    }
  } finally {
    goog.moduleLoaderState_ = previousState;
  }
};


/**
 * @private @const
 */
goog.loadModuleFromSource_ = /** @type {function(string):?} */ (function() {
  // NOTE: we avoid declaring parameters or local variables here to avoid
  // masking globals or leaking values into the module definition.
  'use strict';
  var exports = {};
  eval(arguments[0]);
  return exports;
});


/**
 * Normalize a file path by removing redundant ".." and extraneous "." file
 * path components.
 * @param {string} path
 * @return {string}
 * @private
 */
goog.normalizePath_ = function(path) {
  var components = path.split('/');
  var i = 0;
  while (i < components.length) {
    if (components[i] == '.') {
      components.splice(i, 1);
    } else if (
        i && components[i] == '..' && components[i - 1] &&
        components[i - 1] != '..') {
      components.splice(--i, 2);
    } else {
      i++;
    }
  }
  return components.join('/');
};


/**
 * Provides a hook for loading a file when using Closure's goog.require() API
 * with goog.modules.  In particular this hook is provided to support Node.js.
 *
 * @type {(function(string):string)|undefined}
 */
goog.global.CLOSURE_LOAD_FILE_SYNC;


/**
 * Loads file by synchronous XHR. Should not be used in production environments.
 * @param {string} src Source URL.
 * @return {?string} File contents, or null if load failed.
 * @private
 */
goog.loadFileSync_ = function(src) {
  if (goog.global.CLOSURE_LOAD_FILE_SYNC) {
    return goog.global.CLOSURE_LOAD_FILE_SYNC(src);
  } else {
    try {
      /** @type {XMLHttpRequest} */
      var xhr = new goog.global['XMLHttpRequest']();
      xhr.open('get', src, false);
      xhr.send();
      // NOTE: Successful http: requests have a status of 200, but successful
      // file: requests may have a status of zero.  Any other status, or a
      // thrown exception (particularly in case of file: requests) indicates
      // some sort of error, which we treat as a missing or unavailable file.
      return xhr.status == 0 || xhr.status == 200 ? xhr.responseText : null;
    } catch (err) {
      // No need to rethrow or log, since errors should show up on their own.
      return null;
    }
  }
};


/**
 * Lazily retrieves the transpiler and applies it to the source.
 * @param {string} code JS code.
 * @param {string} path Path to the code.
 * @param {string} target Language level output.
 * @return {string} The transpiled code.
 * @private
 */
goog.transpile_ = function(code, path, target) {
  var jscomp = goog.global['$jscomp'];
  if (!jscomp) {
    goog.global['$jscomp'] = jscomp = {};
  }
  var transpile = jscomp.transpile;
  if (!transpile) {
    var transpilerPath = goog.basePath + goog.TRANSPILER;
    var transpilerCode = goog.loadFileSync_(transpilerPath);
    if (transpilerCode) {
      // This must be executed synchronously, since by the time we know we
      // need it, we're about to load and write the ES6 code synchronously,
      // so a normal script-tag load will be too slow. Wrapped in a function
      // so that code is eval'd in the global scope.
      (function() {
        (0, eval)(transpilerCode + '\n//# sourceURL=' + transpilerPath);
      }).call(goog.global);
      // Even though the transpiler is optional, if $gwtExport is found, it's
      // a sign the transpiler was loaded and the $jscomp.transpile *should*
      // be there.
      if (goog.global['$gwtExport'] && goog.global['$gwtExport']['$jscomp'] &&
          !goog.global['$gwtExport']['$jscomp']['transpile']) {
        throw new Error(
            'The transpiler did not properly export the "transpile" ' +
            'method. $gwtExport: ' + JSON.stringify(goog.global['$gwtExport']));
      }
      // transpile.js only exports a single $jscomp function, transpile. We
      // grab just that and add it to the existing definition of $jscomp which
      // contains the polyfills.
      goog.global['$jscomp'].transpile =
          goog.global['$gwtExport']['$jscomp']['transpile'];
      jscomp = goog.global['$jscomp'];
      transpile = jscomp.transpile;
    }
  }
  if (!transpile) {
    // The transpiler is an optional component.  If it's not available then
    // replace it with a pass-through function that simply logs.
    var suffix = ' requires transpilation but no transpiler was found.';
    // MOE:begin_strip
    suffix +=  // Provide a more appropriate message internally.
        ' Please add "//javascript/closure:transpiler" as a data ' +
        'dependency to ensure it is included.';
    // MOE:end_strip
    transpile = jscomp.transpile = function(code, path) {
      // TODO(sdh): figure out some way to get this error to show up
      // in test results, noting that the failure may occur in many
      // different ways, including in loadModule() before the test
      // runner even comes up.
      goog.logToConsole_(path + suffix);
      return code;
    };
  }
  // Note: any transpilation errors/warnings will be logged to the console.
  return transpile(code, path, target);
};

//==============================================================================
// Language Enhancements
//==============================================================================


/**
 * This is a "fixed" version of the typeof operator.  It differs from the typeof
 * operator in such a way that null returns 'null' and arrays return 'array'.
 * @param {?} value The value to get the type of.
 * @return {string} The name of the type.
 */
goog.typeOf = function(value) {
  var s = typeof value;
  if (s == 'object') {
    if (value) {
      // Check these first, so we can avoid calling Object.prototype.toString if
      // possible.
      //
      // IE9 and below improperly marshals typeof across execution contexts, but
      // a cross-context object will still return false for "instanceof Object".
      if (value instanceof Array) {
        return 'array';
      } else if (value instanceof Object) {
        return s;
      }

      // HACK: In order to use an Object prototype method on the arbitrary
      //   value, the compiler requires the value be cast to type Object,
      //   even though the ECMA spec explicitly allows it.
      var className = Object.prototype.toString.call(
          /** @type {!Object} */ (value));
      // In Firefox 3.6, attempting to access iframe window objects' length
      // property throws an NS_ERROR_FAILURE, so we need to special-case it
      // here.
      if (className == '[object Window]') {
        return 'object';
      }

      // We cannot always use constructor == Array or instanceof Array because
      // different frames have different Array objects. In IE6, if the iframe
      // where the array was created is destroyed, the array loses its
      // prototype. Then dereferencing val.splice here throws an exception, so
      // we can't use goog.isFunction. Calling typeof directly returns 'unknown'
      // so that will work. In this case, this function will return false and
      // most array functions will still work because the array is still
      // array-like (supports length and []) even though it has lost its
      // prototype.
      // Mark Miller noticed that Object.prototype.toString
      // allows access to the unforgeable [[Class]] property.
      //  15.2.4.2 Object.prototype.toString ( )
      //  When the toString method is called, the following steps are taken:
      //      1. Get the [[Class]] property of this object.
      //      2. Compute a string value by concatenating the three strings
      //         "[object ", Result(1), and "]".
      //      3. Return Result(2).
      // and this behavior survives the destruction of the execution context.
      if ((className == '[object Array]' ||
           // In IE all non value types are wrapped as objects across window
           // boundaries (not iframe though) so we have to do object detection
           // for this edge case.
           typeof value.length == 'number' &&
               typeof value.splice != 'undefined' &&
               typeof value.propertyIsEnumerable != 'undefined' &&
               !value.propertyIsEnumerable('splice')

               )) {
        return 'array';
      }
      // HACK: There is still an array case that fails.
      //     function ArrayImpostor() {}
      //     ArrayImpostor.prototype = [];
      //     var impostor = new ArrayImpostor;
      // this can be fixed by getting rid of the fast path
      // (value instanceof Array) and solely relying on
      // (value && Object.prototype.toString.vall(value) === '[object Array]')
      // but that would require many more function calls and is not warranted
      // unless closure code is receiving objects from untrusted sources.

      // IE in cross-window calls does not correctly marshal the function type
      // (it appears just as an object) so we cannot use just typeof val ==
      // 'function'. However, if the object has a call property, it is a
      // function.
      if ((className == '[object Function]' ||
           typeof value.call != 'undefined' &&
               typeof value.propertyIsEnumerable != 'undefined' &&
               !value.propertyIsEnumerable('call'))) {
        return 'function';
      }

    } else {
      return 'null';
    }

  } else if (s == 'function' && typeof value.call == 'undefined') {
    // In Safari typeof nodeList returns 'function', and on Firefox typeof
    // behaves similarly for HTML{Applet,Embed,Object}, Elements and RegExps. We
    // would like to return object for those and we can detect an invalid
    // function by making sure that the function object has a call method.
    return 'object';
  }
  return s;
};


/**
 * Returns true if the specified value is an array.
 * @param {?} val Variable to test.
 * @return {boolean} Whether variable is an array.
 * @deprecated Use Array.isArray instead.
 */
goog.isArray = function(val) {
  return goog.typeOf(val) == 'array';
};


/**
 * Returns true if the object looks like an array. To qualify as array like
 * the value needs to be either a NodeList or an object with a Number length
 * property. Note that for this function neither strings nor functions are
 * considered "array-like".
 *
 * @param {?} val Variable to test.
 * @return {boolean} Whether variable is an array.
 */
goog.isArrayLike = function(val) {
  var type = goog.typeOf(val);
  // We do not use goog.isObject here in order to exclude function values.
  return type == 'array' || type == 'object' && typeof val.length == 'number';
};


/**
 * Returns true if the object looks like a Date. To qualify as Date-like the
 * value needs to be an object and have a getFullYear() function.
 * @param {?} val Variable to test.
 * @return {boolean} Whether variable is a like a Date.
 */
goog.isDateLike = function(val) {
  return goog.isObject(val) && typeof val.getFullYear == 'function';
};


/**
 * Returns true if the specified value is a function.
 * @param {?} val Variable to test.
 * @return {boolean} Whether variable is a function.
 */
goog.isFunction = function(val) {
  return goog.typeOf(val) == 'function';
};


/**
 * Returns true if the specified value is an object.  This includes arrays and
 * functions.
 * @param {?} val Variable to test.
 * @return {boolean} Whether variable is an object.
 */
goog.isObject = function(val) {
  var type = typeof val;
  return type == 'object' && val != null || type == 'function';
  // return Object(val) === val also works, but is slower, especially if val is
  // not an object.
};


/**
 * Gets a unique ID for an object. This mutates the object so that further calls
 * with the same object as a parameter returns the same value. The unique ID is
 * guaranteed to be unique across the current session amongst objects that are
 * passed into `getUid`. There is no guarantee that the ID is unique or
 * consistent across sessions. It is unsafe to generate unique ID for function
 * prototypes.
 *
 * @param {Object} obj The object to get the unique ID for.
 * @return {number} The unique ID for the object.
 */
goog.getUid = function(obj) {
  // TODO(arv): Make the type stricter, do not accept null.
  return Object.prototype.hasOwnProperty.call(obj, goog.UID_PROPERTY_) &&
      obj[goog.UID_PROPERTY_] ||
      (obj[goog.UID_PROPERTY_] = ++goog.uidCounter_);
};


/**
 * Whether the given object is already assigned a unique ID.
 *
 * This does not modify the object.
 *
 * @param {!Object} obj The object to check.
 * @return {boolean} Whether there is an assigned unique id for the object.
 */
goog.hasUid = function(obj) {
  return !!obj[goog.UID_PROPERTY_];
};


/**
 * Removes the unique ID from an object. This is useful if the object was
 * previously mutated using `goog.getUid` in which case the mutation is
 * undone.
 * @param {Object} obj The object to remove the unique ID field from.
 */
goog.removeUid = function(obj) {
  // TODO(arv): Make the type stricter, do not accept null.

  // In IE, DOM nodes are not instances of Object and throw an exception if we
  // try to delete.  Instead we try to use removeAttribute.
  if (obj !== null && 'removeAttribute' in obj) {
    obj.removeAttribute(goog.UID_PROPERTY_);
  }

  try {
    delete obj[goog.UID_PROPERTY_];
  } catch (ex) {
  }
};


/**
 * Name for unique ID property. Initialized in a way to help avoid collisions
 * with other closure JavaScript on the same page.
 * @type {string}
 * @private
 */
goog.UID_PROPERTY_ = 'closure_uid_' + ((Math.random() * 1e9) >>> 0);


/**
 * Counter for UID.
 * @type {number}
 * @private
 */
goog.uidCounter_ = 0;


/**
 * Adds a hash code field to an object. The hash code is unique for the
 * given object.
 * @param {Object} obj The object to get the hash code for.
 * @return {number} The hash code for the object.
 * @deprecated Use goog.getUid instead.
 */
goog.getHashCode = goog.getUid;


/**
 * Removes the hash code field from an object.
 * @param {Object} obj The object to remove the field from.
 * @deprecated Use goog.removeUid instead.
 */
goog.removeHashCode = goog.removeUid;


/**
 * Clones a value. The input may be an Object, Array, or basic type. Objects and
 * arrays will be cloned recursively.
 *
 * WARNINGS:
 * <code>goog.cloneObject</code> does not detect reference loops. Objects that
 * refer to themselves will cause infinite recursion.
 *
 * <code>goog.cloneObject</code> is unaware of unique identifiers, and copies
 * UIDs created by <code>getUid</code> into cloned results.
 *
 * @param {*} obj The value to clone.
 * @return {*} A clone of the input value.
 * @deprecated goog.cloneObject is unsafe. Prefer the goog.object methods.
 */
goog.cloneObject = function(obj) {
  var type = goog.typeOf(obj);
  if (type == 'object' || type == 'array') {
    if (typeof obj.clone === 'function') {
      return obj.clone();
    }
    var clone = type == 'array' ? [] : {};
    for (var key in obj) {
      clone[key] = goog.cloneObject(obj[key]);
    }
    return clone;
  }

  return obj;
};


/**
 * A native implementation of goog.bind.
 * @param {?function(this:T, ...)} fn A function to partially apply.
 * @param {T} selfObj Specifies the object which this should point to when the
 *     function is run.
 * @param {...*} var_args Additional arguments that are partially applied to the
 *     function.
 * @return {!Function} A partially-applied form of the function goog.bind() was
 *     invoked as a method of.
 * @template T
 * @private
 */
goog.bindNative_ = function(fn, selfObj, var_args) {
  return /** @type {!Function} */ (fn.call.apply(fn.bind, arguments));
};


/**
 * A pure-JS implementation of goog.bind.
 * @param {?function(this:T, ...)} fn A function to partially apply.
 * @param {T} selfObj Specifies the object which this should point to when the
 *     function is run.
 * @param {...*} var_args Additional arguments that are partially applied to the
 *     function.
 * @return {!Function} A partially-applied form of the function goog.bind() was
 *     invoked as a method of.
 * @template T
 * @private
 */
goog.bindJs_ = function(fn, selfObj, var_args) {
  if (!fn) {
    throw new Error();
  }

  if (arguments.length > 2) {
    var boundArgs = Array.prototype.slice.call(arguments, 2);
    return function() {
      // Prepend the bound arguments to the current arguments.
      var newArgs = Array.prototype.slice.call(arguments);
      Array.prototype.unshift.apply(newArgs, boundArgs);
      return fn.apply(selfObj, newArgs);
    };

  } else {
    return function() {
      return fn.apply(selfObj, arguments);
    };
  }
};


/**
 * Partially applies this function to a particular 'this object' and zero or
 * more arguments. The result is a new function with some arguments of the first
 * function pre-filled and the value of this 'pre-specified'.
 *
 * Remaining arguments specified at call-time are appended to the pre-specified
 * ones.
 *
 * Also see: {@link #partial}.
 *
 * Usage:
 * <pre>var barMethBound = goog.bind(myFunction, myObj, 'arg1', 'arg2');
 * barMethBound('arg3', 'arg4');</pre>
 *
 * @param {?function(this:T, ...)} fn A function to partially apply.
 * @param {T} selfObj Specifies the object which this should point to when the
 *     function is run.
 * @param {...*} var_args Additional arguments that are partially applied to the
 *     function.
 * @return {!Function} A partially-applied form of the function goog.bind() was
 *     invoked as a method of.
 * @template T
 * @suppress {deprecated} See above.
 */
goog.bind = function(fn, selfObj, var_args) {
  // TODO(nicksantos): narrow the type signature.
  if (Function.prototype.bind &&
      // NOTE(nicksantos): Somebody pulled base.js into the default Chrome
      // extension environment. This means that for Chrome extensions, they get
      // the implementation of Function.prototype.bind that calls goog.bind
      // instead of the native one. Even worse, we don't want to introduce a
      // circular dependency between goog.bind and Function.prototype.bind, so
      // we have to hack this to make sure it works correctly.
      Function.prototype.bind.toString().indexOf('native code') != -1) {
    goog.bind = goog.bindNative_;
  } else {
    goog.bind = goog.bindJs_;
  }
  return goog.bind.apply(null, arguments);
};


/**
 * Like goog.bind(), except that a 'this object' is not required. Useful when
 * the target function is already bound.
 *
 * Usage:
 * var g = goog.partial(f, arg1, arg2);
 * g(arg3, arg4);
 *
 * @param {Function} fn A function to partially apply.
 * @param {...*} var_args Additional arguments that are partially applied to fn.
 * @return {!Function} A partially-applied form of the function goog.partial()
 *     was invoked as a method of.
 */
goog.partial = function(fn, var_args) {
  var args = Array.prototype.slice.call(arguments, 1);
  return function() {
    // Clone the array (with slice()) and append additional arguments
    // to the existing arguments.
    var newArgs = args.slice();
    newArgs.push.apply(newArgs, arguments);
    return fn.apply(/** @type {?} */ (this), newArgs);
  };
};


/**
 * Copies all the members of a source object to a target object. This method
 * does not work on all browsers for all objects that contain keys such as
 * toString or hasOwnProperty. Use goog.object.extend for this purpose.
 *
 * NOTE: Some have advocated for the use of goog.mixin to setup classes
 * with multiple inheritence (traits, mixins, etc).  However, as it simply
 * uses "for in", this is not compatible with ES6 classes whose methods are
 * non-enumerable.  Changing this, would break cases where non-enumerable
 * properties are not expected.
 *
 * @param {Object} target Target.
 * @param {Object} source Source.
 * @deprecated Prefer Object.assign
 */
goog.mixin = function(target, source) {
  for (var x in source) {
    target[x] = source[x];
  }

  // For IE7 or lower, the for-in-loop does not contain any properties that are
  // not enumerable on the prototype object (for example, isPrototypeOf from
  // Object.prototype) but also it will not include 'replace' on objects that
  // extend String and change 'replace' (not that it is common for anyone to
  // extend anything except Object).
};


/**
 * @return {number} An integer value representing the number of milliseconds
 *     between midnight, January 1, 1970 and the current time.
 * @deprecated Use Date.now
 */
goog.now = (goog.TRUSTED_SITE && Date.now) || (function() {
             // Unary plus operator converts its operand to a number which in
             // the case of
             // a date is done by calling getTime().
             return +new Date();
           });


/**
 * Evals JavaScript in the global scope.  In IE this uses execScript, other
 * browsers use goog.global.eval. If goog.global.eval does not evaluate in the
 * global scope (for example, in Safari), appends a script tag instead.
 * Throws an exception if neither execScript or eval is defined.
 * @param {string} script JavaScript string.
 */
goog.globalEval = function(script) {
  if (goog.global.execScript) {
    goog.global.execScript(script, 'JavaScript');
  } else if (goog.global.eval) {
    // Test to see if eval works
    if (goog.evalWorksForGlobals_ == null) {
      try {
        goog.global.eval('var _evalTest_ = 1;');
      } catch (ignore) {
      }
      if (typeof goog.global['_evalTest_'] != 'undefined') {
        try {
          delete goog.global['_evalTest_'];
        } catch (ignore) {
          // Microsoft edge fails the deletion above in strict mode.
        }
        goog.evalWorksForGlobals_ = true;
      } else {
        goog.evalWorksForGlobals_ = false;
      }
    }

    if (goog.evalWorksForGlobals_) {
      goog.global.eval(script);
    } else {
      /** @type {!Document} */
      var doc = goog.global.document;
      var scriptElt =
          /** @type {!HTMLScriptElement} */ (doc.createElement('script'));
      scriptElt.type = 'text/javascript';
      scriptElt.defer = false;
      // Note(pupius): can't use .innerHTML since "t('<test>')" will fail and
      // .text doesn't work in Safari 2.  Therefore we append a text node.
      scriptElt.appendChild(doc.createTextNode(script));
      doc.head.appendChild(scriptElt);
      doc.head.removeChild(scriptElt);
    }
  } else {
    throw new Error('goog.globalEval not available');
  }
};


/**
 * Indicates whether or not we can call 'eval' directly to eval code in the
 * global scope. Set to a Boolean by the first call to goog.globalEval (which
 * empirically tests whether eval works for globals). @see goog.globalEval
 * @type {?boolean}
 * @private
 */
goog.evalWorksForGlobals_ = null;


/**
 * Optional map of CSS class names to obfuscated names used with
 * goog.getCssName().
 * @private {!Object<string, string>|undefined}
 * @see goog.setCssNameMapping
 */
goog.cssNameMapping_;


/**
 * Optional obfuscation style for CSS class names. Should be set to either
 * 'BY_WHOLE' or 'BY_PART' if defined.
 * @type {string|undefined}
 * @private
 * @see goog.setCssNameMapping
 */
goog.cssNameMappingStyle_;



/**
 * A hook for modifying the default behavior goog.getCssName. The function
 * if present, will receive the standard output of the goog.getCssName as
 * its input.
 *
 * @type {(function(string):string)|undefined}
 */
goog.global.CLOSURE_CSS_NAME_MAP_FN;


/**
 * Handles strings that are intended to be used as CSS class names.
 *
 * This function works in tandem with @see goog.setCssNameMapping.
 *
 * Without any mapping set, the arguments are simple joined with a hyphen and
 * passed through unaltered.
 *
 * When there is a mapping, there are two possible styles in which these
 * mappings are used. In the BY_PART style, each part (i.e. in between hyphens)
 * of the passed in css name is rewritten according to the map. In the BY_WHOLE
 * style, the full css name is looked up in the map directly. If a rewrite is
 * not specified by the map, the compiler will output a warning.
 *
 * When the mapping is passed to the compiler, it will replace calls to
 * goog.getCssName with the strings from the mapping, e.g.
 *     var x = goog.getCssName('foo');
 *     var y = goog.getCssName(this.baseClass, 'active');
 *  becomes:
 *     var x = 'foo';
 *     var y = this.baseClass + '-active';
 *
 * If one argument is passed it will be processed, if two are passed only the
 * modifier will be processed, as it is assumed the first argument was generated
 * as a result of calling goog.getCssName.
 *
 * @param {string} className The class name.
 * @param {string=} opt_modifier A modifier to be appended to the class name.
 * @return {string} The class name or the concatenation of the class name and
 *     the modifier.
 */
goog.getCssName = function(className, opt_modifier) {
  // String() is used for compatibility with compiled soy where the passed
  // className can be non-string objects.
  if (String(className).charAt(0) == '.') {
    throw new Error(
        'className passed in goog.getCssName must not start with ".".' +
        ' You passed: ' + className);
  }

  var getMapping = function(cssName) {
    return goog.cssNameMapping_[cssName] || cssName;
  };

  var renameByParts = function(cssName) {
    // Remap all the parts individually.
    var parts = cssName.split('-');
    var mapped = [];
    for (var i = 0; i < parts.length; i++) {
      mapped.push(getMapping(parts[i]));
    }
    return mapped.join('-');
  };

  var rename;
  if (goog.cssNameMapping_) {
    rename =
        goog.cssNameMappingStyle_ == 'BY_WHOLE' ? getMapping : renameByParts;
  } else {
    rename = function(a) {
      return a;
    };
  }

  var result =
      opt_modifier ? className + '-' + rename(opt_modifier) : rename(className);

  // The special CLOSURE_CSS_NAME_MAP_FN allows users to specify further
  // processing of the class name.
  if (goog.global.CLOSURE_CSS_NAME_MAP_FN) {
    return goog.global.CLOSURE_CSS_NAME_MAP_FN(result);
  }

  return result;
};


/**
 * Sets the map to check when returning a value from goog.getCssName(). Example:
 * <pre>
 * goog.setCssNameMapping({
 *   "goog": "a",
 *   "disabled": "b",
 * });
 *
 * var x = goog.getCssName('goog');
 * // The following evaluates to: "a a-b".
 * goog.getCssName('goog') + ' ' + goog.getCssName(x, 'disabled')
 * </pre>
 * When declared as a map of string literals to string literals, the JSCompiler
 * will replace all calls to goog.getCssName() using the supplied map if the
 * --process_closure_primitives flag is set.
 *
 * @param {!Object} mapping A map of strings to strings where keys are possible
 *     arguments to goog.getCssName() and values are the corresponding values
 *     that should be returned.
 * @param {string=} opt_style The style of css name mapping. There are two valid
 *     options: 'BY_PART', and 'BY_WHOLE'.
 * @see goog.getCssName for a description.
 */
goog.setCssNameMapping = function(mapping, opt_style) {
  goog.cssNameMapping_ = mapping;
  goog.cssNameMappingStyle_ = opt_style;
};


/**
 * To use CSS renaming in compiled mode, one of the input files should have a
 * call to goog.setCssNameMapping() with an object literal that the JSCompiler
 * can extract and use to replace all calls to goog.getCssName(). In uncompiled
 * mode, JavaScript code should be loaded before this base.js file that declares
 * a global variable, CLOSURE_CSS_NAME_MAPPING, which is used below. This is
 * to ensure that the mapping is loaded before any calls to goog.getCssName()
 * are made in uncompiled mode.
 *
 * A hook for overriding the CSS name mapping.
 * @type {!Object<string, string>|undefined}
 */
goog.global.CLOSURE_CSS_NAME_MAPPING;


if (!COMPILED && goog.global.CLOSURE_CSS_NAME_MAPPING) {
  // This does not call goog.setCssNameMapping() because the JSCompiler
  // requires that goog.setCssNameMapping() be called with an object literal.
  goog.cssNameMapping_ = goog.global.CLOSURE_CSS_NAME_MAPPING;
}


/**
 * Gets a localized message.
 *
 * This function is a compiler primitive. If you give the compiler a localized
 * message bundle, it will replace the string at compile-time with a localized
 * version, and expand goog.getMsg call to a concatenated string.
 *
 * Messages must be initialized in the form:
 * <code>
 * var MSG_NAME = goog.getMsg('Hello {$placeholder}', {'placeholder': 'world'});
 * </code>
 *
 * This function produces a string which should be treated as plain text. Use
 * {@link goog.html.SafeHtmlFormatter} in conjunction with goog.getMsg to
 * produce SafeHtml.
 *
 * @param {string} str Translatable string, places holders in the form {$foo}.
 * @param {Object<string, string>=} opt_values Maps place holder name to value.
 * @param {{html: boolean}=} opt_options Options:
 *     html: Escape '<' in str to '&lt;'. Used by Closure Templates where the
 *     generated code size and performance is critical which is why {@link
 *     goog.html.SafeHtmlFormatter} is not used. The value must be literal true
 *     or false.
 * @return {string} message with placeholders filled.
 */
goog.getMsg = function(str, opt_values, opt_options) {
  if (opt_options && opt_options.html) {
    // Note that '&' is not replaced because the translation can contain HTML
    // entities.
    str = str.replace(/</g, '&lt;');
  }
  if (opt_values) {
    str = str.replace(/\{\$([^}]+)}/g, function(match, key) {
      return (opt_values != null && key in opt_values) ? opt_values[key] :
                                                         match;
    });
  }
  return str;
};


/**
 * Gets a localized message. If the message does not have a translation, gives a
 * fallback message.
 *
 * This is useful when introducing a new message that has not yet been
 * translated into all languages.
 *
 * This function is a compiler primitive. Must be used in the form:
 * <code>var x = goog.getMsgWithFallback(MSG_A, MSG_B);</code>
 * where MSG_A and MSG_B were initialized with goog.getMsg.
 *
 * @param {string} a The preferred message.
 * @param {string} b The fallback message.
 * @return {string} The best translated message.
 */
goog.getMsgWithFallback = function(a, b) {
  return a;
};


/**
 * Exposes an unobfuscated global namespace path for the given object.
 * Note that fields of the exported object *will* be obfuscated, unless they are
 * exported in turn via this function or goog.exportProperty.
 *
 * Also handy for making public items that are defined in anonymous closures.
 *
 * ex. goog.exportSymbol('public.path.Foo', Foo);
 *
 * ex. goog.exportSymbol('public.path.Foo.staticFunction', Foo.staticFunction);
 *     public.path.Foo.staticFunction();
 *
 * ex. goog.exportSymbol('public.path.Foo.prototype.myMethod',
 *                       Foo.prototype.myMethod);
 *     new public.path.Foo().myMethod();
 *
 * @param {string} publicPath Unobfuscated name to export.
 * @param {*} object Object the name should point to.
 * @param {Object=} opt_objectToExportTo The object to add the path to; default
 *     is goog.global.
 */
goog.exportSymbol = function(publicPath, object, opt_objectToExportTo) {
  goog.exportPath_(publicPath, object, opt_objectToExportTo);
};


/**
 * Exports a property unobfuscated into the object's namespace.
 * ex. goog.exportProperty(Foo, 'staticFunction', Foo.staticFunction);
 * ex. goog.exportProperty(Foo.prototype, 'myMethod', Foo.prototype.myMethod);
 * @param {Object} object Object whose static property is being exported.
 * @param {string} publicName Unobfuscated name to export.
 * @param {*} symbol Object the name should point to.
 */
goog.exportProperty = function(object, publicName, symbol) {
  object[publicName] = symbol;
};


/**
 * Inherit the prototype methods from one constructor into another.
 *
 * Usage:
 * <pre>
 * function ParentClass(a, b) { }
 * ParentClass.prototype.foo = function(a) { };
 *
 * function ChildClass(a, b, c) {
 *   ChildClass.base(this, 'constructor', a, b);
 * }
 * goog.inherits(ChildClass, ParentClass);
 *
 * var child = new ChildClass('a', 'b', 'see');
 * child.foo(); // This works.
 * </pre>
 *
 * @param {!Function} childCtor Child class.
 * @param {!Function} parentCtor Parent class.
 * @suppress {strictMissingProperties} superClass_ and base is not defined on
 *    Function.
 */
goog.inherits = function(childCtor, parentCtor) {
  /** @constructor */
  function tempCtor() {}
  tempCtor.prototype = parentCtor.prototype;
  childCtor.superClass_ = parentCtor.prototype;
  childCtor.prototype = new tempCtor();
  /** @override */
  childCtor.prototype.constructor = childCtor;

  /**
   * Calls superclass constructor/method.
   *
   * This function is only available if you use goog.inherits to
   * express inheritance relationships between classes.
   *
   * NOTE: This is a replacement for goog.base and for superClass_
   * property defined in childCtor.
   *
   * @param {!Object} me Should always be "this".
   * @param {string} methodName The method name to call. Calling
   *     superclass constructor can be done with the special string
   *     'constructor'.
   * @param {...*} var_args The arguments to pass to superclass
   *     method/constructor.
   * @return {*} The return value of the superclass method/constructor.
   */
  childCtor.base = function(me, methodName, var_args) {
    // Copying using loop to avoid deop due to passing arguments object to
    // function. This is faster in many JS engines as of late 2014.
    var args = new Array(arguments.length - 2);
    for (var i = 2; i < arguments.length; i++) {
      args[i - 2] = arguments[i];
    }
    return parentCtor.prototype[methodName].apply(me, args);
  };
};


/**
 * Allow for aliasing within scope functions.  This function exists for
 * uncompiled code - in compiled code the calls will be inlined and the aliases
 * applied.  In uncompiled code the function is simply run since the aliases as
 * written are valid JavaScript.
 *
 * MOE:begin_intracomment_strip
 * See the goog.scope document at http://go/goog.scope
 *
 * For more on goog.scope deprecation, see the style guide entry:
 * http://go/jsstyle#appendices-legacy-exceptions-goog-scope
 * MOE:end_intracomment_strip
 *
 * @param {function()} fn Function to call.  This function can contain aliases
 *     to namespaces (e.g. "var dom = goog.dom") or classes
 *     (e.g. "var Timer = goog.Timer").
 * @deprecated Use goog.module instead.
 */
goog.scope = function(fn) {
  if (goog.isInModuleLoader_()) {
    throw new Error('goog.scope is not supported within a module.');
  }
  fn.call(goog.global);
};


/*
 * To support uncompiled, strict mode bundles that use eval to divide source
 * like so:
 *    eval('someSource;//# sourceUrl sourcefile.js');
 * We need to export the globally defined symbols "goog" and "COMPILED".
 * Exporting "goog" breaks the compiler optimizations, so we required that
 * be defined externally.
 * NOTE: We don't use goog.exportSymbol here because we don't want to trigger
 * extern generation when that compiler option is enabled.
 */
if (!COMPILED) {
  goog.global['COMPILED'] = COMPILED;
}


//==============================================================================
// goog.defineClass implementation
//==============================================================================


/**
 * Creates a restricted form of a Closure "class":
 *   - from the compiler's perspective, the instance returned from the
 *     constructor is sealed (no new properties may be added).  This enables
 *     better checks.
 *   - the compiler will rewrite this definition to a form that is optimal
 *     for type checking and optimization (initially this will be a more
 *     traditional form).
 *
 * @param {Function} superClass The superclass, Object or null.
 * @param {goog.defineClass.ClassDescriptor} def
 *     An object literal describing
 *     the class.  It may have the following properties:
 *     "constructor": the constructor function
 *     "statics": an object literal containing methods to add to the constructor
 *        as "static" methods or a function that will receive the constructor
 *        function as its only parameter to which static properties can
 *        be added.
 *     all other properties are added to the prototype.
 * @return {!Function} The class constructor.
 * @deprecated Use ES6 class syntax instead.
 */
goog.defineClass = function(superClass, def) {
  // TODO(johnlenz): consider making the superClass an optional parameter.
  var constructor = def.constructor;
  var statics = def.statics;
  // Wrap the constructor prior to setting up the prototype and static methods.
  if (!constructor || constructor == Object.prototype.constructor) {
    constructor = function() {
      throw new Error(
          'cannot instantiate an interface (no constructor defined).');
    };
  }

  var cls = goog.defineClass.createSealingConstructor_(constructor, superClass);
  if (superClass) {
    goog.inherits(cls, superClass);
  }

  // Remove all the properties that should not be copied to the prototype.
  delete def.constructor;
  delete def.statics;

  goog.defineClass.applyProperties_(cls.prototype, def);
  if (statics != null) {
    if (statics instanceof Function) {
      statics(cls);
    } else {
      goog.defineClass.applyProperties_(cls, statics);
    }
  }

  return cls;
};


/**
 * @typedef {{
 *   constructor: (!Function|undefined),
 *   statics: (Object|undefined|function(Function):void)
 * }}
 */
goog.defineClass.ClassDescriptor;


/**
 * @define {boolean} Whether the instances returned by goog.defineClass should
 *     be sealed when possible.
 *
 * When sealing is disabled the constructor function will not be wrapped by
 * goog.defineClass, making it incompatible with ES6 class methods.
 */
goog.defineClass.SEAL_CLASS_INSTANCES =
    goog.define('goog.defineClass.SEAL_CLASS_INSTANCES', goog.DEBUG);


/**
 * If goog.defineClass.SEAL_CLASS_INSTANCES is enabled and Object.seal is
 * defined, this function will wrap the constructor in a function that seals the
 * results of the provided constructor function.
 *
 * @param {!Function} ctr The constructor whose results maybe be sealed.
 * @param {Function} superClass The superclass constructor.
 * @return {!Function} The replacement constructor.
 * @private
 */
goog.defineClass.createSealingConstructor_ = function(ctr, superClass) {
  if (!goog.defineClass.SEAL_CLASS_INSTANCES) {
    // Do now wrap the constructor when sealing is disabled. Angular code
    // depends on this for injection to work properly.
    return ctr;
  }

  // Compute whether the constructor is sealable at definition time, rather
  // than when the instance is being constructed.
  var superclassSealable = !goog.defineClass.isUnsealable_(superClass);

  /**
   * @this {Object}
   * @return {?}
   */
  var wrappedCtr = function() {
    // Don't seal an instance of a subclass when it calls the constructor of
    // its super class as there is most likely still setup to do.
    var instance = ctr.apply(this, arguments) || this;
    instance[goog.UID_PROPERTY_] = instance[goog.UID_PROPERTY_];

    if (this.constructor === wrappedCtr && superclassSealable &&
        Object.seal instanceof Function) {
      Object.seal(instance);
    }
    return instance;
  };

  return wrappedCtr;
};


/**
 * @param {Function} ctr The constructor to test.
 * @return {boolean} Whether the constructor has been tagged as unsealable
 *     using goog.tagUnsealableClass.
 * @private
 */
goog.defineClass.isUnsealable_ = function(ctr) {
  return ctr && ctr.prototype &&
      ctr.prototype[goog.UNSEALABLE_CONSTRUCTOR_PROPERTY_];
};


// TODO(johnlenz): share these values with the goog.object
/**
 * The names of the fields that are defined on Object.prototype.
 * @type {!Array<string>}
 * @private
 * @const
 */
goog.defineClass.OBJECT_PROTOTYPE_FIELDS_ = [
  'constructor', 'hasOwnProperty', 'isPrototypeOf', 'propertyIsEnumerable',
  'toLocaleString', 'toString', 'valueOf'
];


// TODO(johnlenz): share this function with the goog.object
/**
 * @param {!Object} target The object to add properties to.
 * @param {!Object} source The object to copy properties from.
 * @private
 */
goog.defineClass.applyProperties_ = function(target, source) {
  // TODO(johnlenz): update this to support ES5 getters/setters

  var key;
  for (key in source) {
    if (Object.prototype.hasOwnProperty.call(source, key)) {
      target[key] = source[key];
    }
  }

  // For IE the for-in-loop does not contain any properties that are not
  // enumerable on the prototype object (for example isPrototypeOf from
  // Object.prototype) and it will also not include 'replace' on objects that
  // extend String and change 'replace' (not that it is common for anyone to
  // extend anything except Object).
  for (var i = 0; i < goog.defineClass.OBJECT_PROTOTYPE_FIELDS_.length; i++) {
    key = goog.defineClass.OBJECT_PROTOTYPE_FIELDS_[i];
    if (Object.prototype.hasOwnProperty.call(source, key)) {
      target[key] = source[key];
    }
  }
};


/**
 * Sealing classes breaks the older idiom of assigning properties on the
 * prototype rather than in the constructor. As such, goog.defineClass
 * must not seal subclasses of these old-style classes until they are fixed.
 * Until then, this marks a class as "broken", instructing defineClass
 * not to seal subclasses.
 * @param {!Function} ctr The legacy constructor to tag as unsealable.
 */
goog.tagUnsealableClass = function(ctr) {
  if (!COMPILED && goog.defineClass.SEAL_CLASS_INSTANCES) {
    ctr.prototype[goog.UNSEALABLE_CONSTRUCTOR_PROPERTY_] = true;
  }
};


/**
 * Name for unsealable tag property.
 * @const @private {string}
 */
goog.UNSEALABLE_CONSTRUCTOR_PROPERTY_ = 'goog_defineClass_legacy_unsealable';


// There's a bug in the compiler where without collapse properties the
// Closure namespace defines do not guard code correctly. To help reduce code
// size also check for !COMPILED even though it redundant until this is fixed.
if (!COMPILED && goog.DEPENDENCIES_ENABLED) {
  // MOE:begin_strip
  // TODO(b/67050526) This object is obsolete but some people are relying on
  // it internally. Keep it around until we migrate them.
  /**
   * @private
   * @type {{
   *   loadFlags: !Object<string, !Object<string, string>>,
   *   nameToPath: !Object<string, string>,
   *   requires: !Object<string, !Object<string, boolean>>,
   *   visited: !Object<string, boolean>,
   *   written: !Object<string, boolean>,
   *   deferred: !Object<string, string>
   * }}
   */
  goog.dependencies_ = {
    loadFlags: {},  // 1 to 1

    nameToPath: {},  // 1 to 1

    requires: {},  // 1 to many

    // Used when resolving dependencies to prevent us from visiting file
    // twice.
    visited: {},

    written: {},  // Used to keep track of script files we have written.

    deferred: {}  // Used to track deferred module evaluations in old IEs
  };

  /**
   * @return {!Object}
   * @private
   */
  goog.getLoader_ = function() {
    return {
      dependencies_: goog.dependencies_,
      writeScriptTag_: goog.writeScriptTag_
    };
  };


  /**
   * @param {string} src The script url.
   * @param {string=} opt_sourceText The optionally source text to evaluate
   * @return {boolean} True if the script was imported, false otherwise.
   * @private
   */
  goog.writeScriptTag_ = function(src, opt_sourceText) {
    if (goog.inHtmlDocument_()) {
      /** @type {!HTMLDocument} */
      var doc = goog.global.document;

      // If the user tries to require a new symbol after document load,
      // something has gone terribly wrong. Doing a document.write would
      // wipe out the page. This does not apply to the CSP-compliant method
      // of writing script tags.
      if (!goog.ENABLE_CHROME_APP_SAFE_SCRIPT_LOADING &&
          doc.readyState == 'complete') {
        // Certain test frameworks load base.js multiple times, which tries
        // to write deps.js each time. If that happens, just fail silently.
        // These frameworks wipe the page between each load of base.js, so this
        // is OK.
        var isDeps = /\bdeps.js$/.test(src);
        if (isDeps) {
          return false;
        } else {
          throw Error('Cannot write "' + src + '" after document load');
        }
      }

      if (opt_sourceText === undefined) {
        var script = '<script type="text/javascript" src="' + src + '"></' +
            'script>';
        doc.write(
            goog.TRUSTED_TYPES_POLICY_ ?
                goog.TRUSTED_TYPES_POLICY_.createHTML(script) :
                script);
      } else {
        var script = '<script type="text/javascript">' +
            goog.protectScriptTag_(opt_sourceText) + '</' +
            'script>';
        doc.write(
            goog.TRUSTED_TYPES_POLICY_ ?
                goog.TRUSTED_TYPES_POLICY_.createHTML(script) :
                script);
      }
      return true;
    } else {
      return false;
    }
  };
  // MOE:end_strip

  /**
   * Tries to detect whether is in the context of an HTML document.
   * @return {boolean} True if it looks like HTML document.
   * @private
   */
  goog.inHtmlDocument_ = function() {
    /** @type {!Document} */
    var doc = goog.global.document;
    return doc != null && 'write' in doc;  // XULDocument misses write.
  };


  /**
   * We'd like to check for if the document readyState is 'loading'; however
   * there are bugs on IE 10 and below where the readyState being anything other
   * than 'complete' is not reliable.
   * @return {boolean}
   * @private
   */
  goog.isDocumentLoading_ = function() {
    // attachEvent is available on IE 6 thru 10 only, and thus can be used to
    // detect those browsers.
    /** @type {!HTMLDocument} */
    var doc = goog.global.document;
    return doc.attachEvent ? doc.readyState != 'complete' :
                             doc.readyState == 'loading';
  };


  /**
   * Tries to detect the base path of base.js script that bootstraps Closure.
   * @private
   */
  goog.findBasePath_ = function() {
    if (goog.global.CLOSURE_BASE_PATH != undefined &&
        // Anti DOM-clobbering runtime check (b/37736576).
        typeof goog.global.CLOSURE_BASE_PATH === 'string') {
      goog.basePath = goog.global.CLOSURE_BASE_PATH;
      return;
    } else if (!goog.inHtmlDocument_()) {
      return;
    }
    /** @type {!Document} */
    var doc = goog.global.document;
    // If we have a currentScript available, use it exclusively.
    var currentScript = doc.currentScript;
    if (currentScript) {
      var scripts = [currentScript];
    } else {
      var scripts = doc.getElementsByTagName('SCRIPT');
    }
    // Search backwards since the current script is in almost all cases the one
    // that has base.js.
    for (var i = scripts.length - 1; i >= 0; --i) {
      var script = /** @type {!HTMLScriptElement} */ (scripts[i]);
      var src = script.src;
      var qmark = src.lastIndexOf('?');
      var l = qmark == -1 ? src.length : qmark;
      if (src.substr(l - 7, 7) == 'base.js') {
        goog.basePath = src.substr(0, l - 7);
        return;
      }
    }
  };

  goog.findBasePath_();

  /** @struct @constructor @final */
  goog.Transpiler = function() {
    /** @private {?Object<string, boolean>} */
    this.requiresTranspilation_ = null;
    /** @private {string} */
    this.transpilationTarget_ = goog.TRANSPILE_TO_LANGUAGE;
  };


  // MOE:begin_strip
  // LINT.IfChange
  // MOE:end_strip
  /**
   * Returns a newly created map from language mode string to a boolean
   * indicating whether transpilation should be done for that mode as well as
   * the highest level language that this environment supports.
   *
   * Guaranteed invariant:
   * For any two modes, l1 and l2 where l2 is a newer mode than l1,
   * `map[l1] == true` implies that `map[l2] == true`.
   *
   * Note this method is extracted and used elsewhere, so it cannot rely on
   * anything external (it should easily be able to be transformed into a
   * standalone, top level function).
   *
   * @private
   * @return {{
   *   target: string,
   *   map: !Object<string, boolean>
   * }}
   */
  goog.Transpiler.prototype.createRequiresTranspilation_ = function() {
    var transpilationTarget = 'es3';
    var /** !Object<string, boolean> */ requiresTranspilation = {'es3': false};
    var transpilationRequiredForAllLaterModes = false;

    /**
     * Adds an entry to requiresTranspliation for the given language mode.
     *
     * IMPORTANT: Calls must be made in order from oldest to newest language
     * mode.
     * @param {string} modeName
     * @param {function(): boolean} isSupported Returns true if the JS engine
     *     supports the given mode.
     */
    function addNewerLanguageTranspilationCheck(modeName, isSupported) {
      if (transpilationRequiredForAllLaterModes) {
        requiresTranspilation[modeName] = true;
      } else if (isSupported()) {
        transpilationTarget = modeName;
        requiresTranspilation[modeName] = false;
      } else {
        requiresTranspilation[modeName] = true;
        transpilationRequiredForAllLaterModes = true;
      }
    }

    /**
     * Does the given code evaluate without syntax errors and return a truthy
     * result?
     */
    function /** boolean */ evalCheck(/** string */ code) {
      try {
        return !!eval(code);
      } catch (ignored) {
        return false;
      }
    }

    var userAgent = goog.global.navigator && goog.global.navigator.userAgent ?
        goog.global.navigator.userAgent :
        '';

    // Identify ES3-only browsers by their incorrect treatment of commas.
    addNewerLanguageTranspilationCheck('es5', function() {
      return evalCheck('[1,].length==1');
    });
    addNewerLanguageTranspilationCheck('es6', function() {
      // Edge has a non-deterministic (i.e., not reproducible) bug with ES6:
      // https://github.com/Microsoft/ChakraCore/issues/1496.
      // MOE:begin_strip
      // TODO(joeltine): Our internal web-testing version of Edge will need to
      // be updated before we can remove this check. See http://b/34945376.
      // MOE:end_strip
      var re = /Edge\/(\d+)(\.\d)*/i;
      var edgeUserAgent = userAgent.match(re);
      if (edgeUserAgent) {
        // The Reflect.construct test below is flaky on Edge. It can sometimes
        // pass or fail on 40 15.15063, so just exit early for Edge and treat
        // it as ES5. Until we're on a more up to date version just always use
        // ES5. See https://github.com/Microsoft/ChakraCore/issues/3217.
        return false;
      }
      // Test es6: [FF50 (?), Edge 14 (?), Chrome 50]
      //   (a) default params (specifically shadowing locals),
      //   (b) destructuring, (c) block-scoped functions,
      //   (d) for-of (const), (e) new.target/Reflect.construct
      var es6fullTest =
          'class X{constructor(){if(new.target!=String)throw 1;this.x=42}}' +
          'let q=Reflect.construct(X,[],String);if(q.x!=42||!(q instanceof ' +
          'String))throw 1;for(const a of[2,3]){if(a==2)continue;function ' +
          'f(z={a}){let a=0;return z.a}{function f(){return 0;}}return f()' +
          '==3}';

      return evalCheck('(()=>{"use strict";' + es6fullTest + '})()');
    });
    // ** and **= are the only new features in 'es7'
    addNewerLanguageTranspilationCheck('es7', function() {
      return evalCheck('2 ** 2 == 4');
    });
    // async functions are the only new features in 'es8'
    addNewerLanguageTranspilationCheck('es8', function() {
      return evalCheck('async () => 1, true');
    });
    addNewerLanguageTranspilationCheck('es9', function() {
      return evalCheck('({...rest} = {}), true');
    });
    addNewerLanguageTranspilationCheck('es_next', function() {
      return false;  // assume it always need to transpile
    });
    return {target: transpilationTarget, map: requiresTranspilation};
  };
  // MOE:begin_strip
  // LINT.ThenChange(//depot/google3/java/com/google/testing/web/devtools/updatebrowserinfo/requires_transpilation.js)
  // MOE:end_strip


  /**
   * Determines whether the given language needs to be transpiled.
   * @param {string} lang
   * @param {string|undefined} module
   * @return {boolean}
   */
  goog.Transpiler.prototype.needsTranspile = function(lang, module) {
    if (goog.TRANSPILE == 'always') {
      return true;
    } else if (goog.TRANSPILE == 'never') {
      return false;
    } else if (!this.requiresTranspilation_) {
      var obj = this.createRequiresTranspilation_();
      this.requiresTranspilation_ = obj.map;
      this.transpilationTarget_ = this.transpilationTarget_ || obj.target;
    }
    if (lang in this.requiresTranspilation_) {
      if (this.requiresTranspilation_[lang]) {
        return true;
      } else if (
          goog.inHtmlDocument_() && module == 'es6' &&
          !('noModule' in goog.global.document.createElement('script'))) {
        return true;
      } else {
        return false;
      }
    } else {
      throw new Error('Unknown language mode: ' + lang);
    }
  };


  /**
   * Lazily retrieves the transpiler and applies it to the source.
   * @param {string} code JS code.
   * @param {string} path Path to the code.
   * @return {string} The transpiled code.
   */
  goog.Transpiler.prototype.transpile = function(code, path) {
    // TODO(johnplaisted): We should delete goog.transpile_ and just have this
    // function. But there's some compile error atm where goog.global is being
    // stripped incorrectly without this.
    return goog.transpile_(code, path, this.transpilationTarget_);
  };


  /** @private @final {!goog.Transpiler} */
  goog.transpiler_ = new goog.Transpiler();

  /**
   * Rewrites closing script tags in input to avoid ending an enclosing script
   * tag.
   *
   * @param {string} str
   * @return {string}
   * @private
   */
  goog.protectScriptTag_ = function(str) {
    return str.replace(/<\/(SCRIPT)/ig, '\\x3c/$1');
  };


  /**
   * A debug loader is responsible for downloading and executing javascript
   * files in an unbundled, uncompiled environment.
   *
   * This can be custimized via the setDependencyFactory method, or by
   * CLOSURE_IMPORT_SCRIPT/CLOSURE_LOAD_FILE_SYNC.
   *
   * @struct @constructor @final @private
   */
  goog.DebugLoader_ = function() {
    /** @private @const {!Object<string, !goog.Dependency>} */
    this.dependencies_ = {};
    /** @private @const {!Object<string, string>} */
    this.idToPath_ = {};
    /** @private @const {!Object<string, boolean>} */
    this.written_ = {};
    /** @private @const {!Array<!goog.Dependency>} */
    this.loadingDeps_ = [];
    /** @private {!Array<!goog.Dependency>} */
    this.depsToLoad_ = [];
    /** @private {boolean} */
    this.paused_ = false;
    /** @private {!goog.DependencyFactory} */
    this.factory_ = new goog.DependencyFactory(goog.transpiler_);
    /** @private @const {!Object<string, !Function>} */
    this.deferredCallbacks_ = {};
    /** @private @const {!Array<string>} */
    this.deferredQueue_ = [];
  };

  /**
   * @param {!Array<string>} namespaces
   * @param {function(): undefined} callback Function to call once all the
   *     namespaces have loaded.
   */
  goog.DebugLoader_.prototype.bootstrap = function(namespaces, callback) {
    var cb = callback;
    function resolve() {
      if (cb) {
        goog.global.setTimeout(cb, 0);
        cb = null;
      }
    }

    if (!namespaces.length) {
      resolve();
      return;
    }

    var deps = [];
    for (var i = 0; i < namespaces.length; i++) {
      var path = this.getPathFromDeps_(namespaces[i]);
      if (!path) {
        throw new Error('Unregonized namespace: ' + namespaces[i]);
      }
      deps.push(this.dependencies_[path]);
    }

    var require = goog.require;
    var loaded = 0;
    for (var i = 0; i < namespaces.length; i++) {
      require(namespaces[i]);
      deps[i].onLoad(function() {
        if (++loaded == namespaces.length) {
          resolve();
        }
      });
    }
  };


  /**
   * Loads the Closure Dependency file.
   *
   * Exposed a public function so CLOSURE_NO_DEPS can be set to false, base
   * loaded, setDependencyFactory called, and then this called. i.e. allows
   * custom loading of the deps file.
   */
  goog.DebugLoader_.prototype.loadClosureDeps = function() {
    // Circumvent addDependency, which would try to transpile deps.js if
    // transpile is set to always.
    var relPath = 'deps.js';
    this.depsToLoad_.push(this.factory_.createDependency(
        goog.normalizePath_(goog.basePath + relPath), relPath, [], [], {},
        false));
    this.loadDeps_();
  };


  /**
   * Notifies the debug loader when a dependency has been requested.
   *
   * @param {string} absPathOrId Path of the dependency or goog id.
   * @param {boolean=} opt_force
   */
  goog.DebugLoader_.prototype.requested = function(absPathOrId, opt_force) {
    var path = this.getPathFromDeps_(absPathOrId);
    if (path &&
        (opt_force || this.areDepsLoaded_(this.dependencies_[path].requires))) {
      var callback = this.deferredCallbacks_[path];
      if (callback) {
        delete this.deferredCallbacks_[path];
        callback();
      }
    }
  };


  /**
   * Sets the dependency factory, which can be used to create custom
   * goog.Dependency implementations to control how dependencies are loaded.
   *
   * @param {!goog.DependencyFactory} factory
   */
  goog.DebugLoader_.prototype.setDependencyFactory = function(factory) {
    this.factory_ = factory;
  };


  /**
   * Travserses the dependency graph and queues the given dependency, and all of
   * its transitive dependencies, for loading and then starts loading if not
   * paused.
   *
   * @param {string} namespace
   * @private
   */
  goog.DebugLoader_.prototype.load_ = function(namespace) {
    if (!this.getPathFromDeps_(namespace)) {
      var errorMessage = 'goog.require could not find: ' + namespace;

      goog.logToConsole_(errorMessage);
      // MOE:begin_strip

      // NOTE(nicksantos): We could always throw an error, but this would
      // break legacy users that depended on this failing silently. Instead,
      // the compiler should warn us when there are invalid goog.require
      // calls. For now, we simply give clients a way to turn strict mode on.
      if (goog.useStrictRequires) {
        throw Error(errorMessage);
      }

      // In external Closure, always error.
      // MOE:end_strip
      // MOE:insert throw Error(errorMessage);
    } else {
      var loader = this;

      var deps = [];

      /** @param {string} namespace */
      var visit = function(namespace) {
        var path = loader.getPathFromDeps_(namespace);

        if (!path) {
          throw new Error('Bad dependency path or symbol: ' + namespace);
        }

        if (loader.written_[path]) {
          return;
        }

        loader.written_[path] = true;

        var dep = loader.dependencies_[path];
        // MOE:begin_strip
        if (goog.dependencies_.written[dep.relativePath]) {
          return;
        }
        // MOE:end_strip
        for (var i = 0; i < dep.requires.length; i++) {
          if (!goog.isProvided_(dep.requires[i])) {
            visit(dep.requires[i]);
          }
        }

        deps.push(dep);
      };

      visit(namespace);

      var wasLoading = !!this.depsToLoad_.length;
      this.depsToLoad_ = this.depsToLoad_.concat(deps);

      if (!this.paused_ && !wasLoading) {
        this.loadDeps_();
      }
    }
  };


  /**
   * Loads any queued dependencies until they are all loaded or paused.
   *
   * @private
   */
  goog.DebugLoader_.prototype.loadDeps_ = function() {
    var loader = this;
    var paused = this.paused_;

    while (this.depsToLoad_.length && !paused) {
      (function() {
        var loadCallDone = false;
        var dep = loader.depsToLoad_.shift();

        var loaded = false;
        loader.loading_(dep);

        var controller = {
          pause: function() {
            if (loadCallDone) {
              throw new Error('Cannot call pause after the call to load.');
            } else {
              paused = true;
            }
          },
          resume: function() {
            if (loadCallDone) {
              loader.resume_();
            } else {
              // Some dep called pause and then resume in the same load call.
              // Just keep running this same loop.
              paused = false;
            }
          },
          loaded: function() {
            if (loaded) {
              throw new Error('Double call to loaded.');
            }

            loaded = true;
            loader.loaded_(dep);
          },
          pending: function() {
            // Defensive copy.
            var pending = [];
            for (var i = 0; i < loader.loadingDeps_.length; i++) {
              pending.push(loader.loadingDeps_[i]);
            }
            return pending;
          },
          /**
           * @param {goog.ModuleType} type
           */
          setModuleState: function(type) {
            goog.moduleLoaderState_ = {
              type: type,
              moduleName: '',
              declareLegacyNamespace: false
            };
          },
          /** @type {function(string, string, string=)} */
          registerEs6ModuleExports: function(
              path, exports, opt_closureNamespace) {
            if (opt_closureNamespace) {
              goog.loadedModules_[opt_closureNamespace] = {
                exports: exports,
                type: goog.ModuleType.ES6,
                moduleId: opt_closureNamespace || ''
              };
            }
          },
          /** @type {function(string, ?)} */
          registerGoogModuleExports: function(moduleId, exports) {
            goog.loadedModules_[moduleId] = {
              exports: exports,
              type: goog.ModuleType.GOOG,
              moduleId: moduleId
            };
          },
          clearModuleState: function() {
            goog.moduleLoaderState_ = null;
          },
          defer: function(callback) {
            if (loadCallDone) {
              throw new Error(
                  'Cannot register with defer after the call to load.');
            }
            loader.defer_(dep, callback);
          },
          areDepsLoaded: function() {
            return loader.areDepsLoaded_(dep.requires);
          }
        };

        try {
          dep.load(controller);
        } finally {
          loadCallDone = true;
        }
      })();
    }

    if (paused) {
      this.pause_();
    }
  };


  /** @private */
  goog.DebugLoader_.prototype.pause_ = function() {
    this.paused_ = true;
  };


  /** @private */
  goog.DebugLoader_.prototype.resume_ = function() {
    if (this.paused_) {
      this.paused_ = false;
      this.loadDeps_();
    }
  };


  /**
   * Marks the given dependency as loading (load has been called but it has not
   * yet marked itself as finished). Useful for dependencies that want to know
   * what else is loading. Example: goog.modules cannot eval if there are
   * loading dependencies.
   *
   * @param {!goog.Dependency} dep
   * @private
   */
  goog.DebugLoader_.prototype.loading_ = function(dep) {
    this.loadingDeps_.push(dep);
  };


  /**
   * Marks the given dependency as having finished loading and being available
   * for require.
   *
   * @param {!goog.Dependency} dep
   * @private
   */
  goog.DebugLoader_.prototype.loaded_ = function(dep) {
    for (var i = 0; i < this.loadingDeps_.length; i++) {
      if (this.loadingDeps_[i] == dep) {
        this.loadingDeps_.splice(i, 1);
        break;
      }
    }

    for (var i = 0; i < this.deferredQueue_.length; i++) {
      if (this.deferredQueue_[i] == dep.path) {
        this.deferredQueue_.splice(i, 1);
        break;
      }
    }

    if (this.loadingDeps_.length == this.deferredQueue_.length &&
        !this.depsToLoad_.length) {
      // Something has asked to load these, but they may not be directly
      // required again later, so load them now that we know we're done loading
      // everything else. e.g. a goog module entry point.
      while (this.deferredQueue_.length) {
        this.requested(this.deferredQueue_.shift(), true);
      }
    }

    dep.loaded();
  };


  /**
   * @param {!Array<string>} pathsOrIds
   * @return {boolean}
   * @private
   */
  goog.DebugLoader_.prototype.areDepsLoaded_ = function(pathsOrIds) {
    for (var i = 0; i < pathsOrIds.length; i++) {
      var path = this.getPathFromDeps_(pathsOrIds[i]);
      if (!path ||
          (!(path in this.deferredCallbacks_) &&
           !goog.isProvided_(pathsOrIds[i]))) {
        return false;
      }
    }

    return true;
  };


  /**
   * @param {string} absPathOrId
   * @return {?string}
   * @private
   */
  goog.DebugLoader_.prototype.getPathFromDeps_ = function(absPathOrId) {
    if (absPathOrId in this.idToPath_) {
      return this.idToPath_[absPathOrId];
    } else if (absPathOrId in this.dependencies_) {
      return absPathOrId;
    } else {
      return null;
    }
  };


  /**
   * @param {!goog.Dependency} dependency
   * @param {!Function} callback
   * @private
   */
  goog.DebugLoader_.prototype.defer_ = function(dependency, callback) {
    this.deferredCallbacks_[dependency.path] = callback;
    this.deferredQueue_.push(dependency.path);
  };


  /**
   * Interface for goog.Dependency implementations to have some control over
   * loading of dependencies.
   *
   * @record
   */
  goog.LoadController = function() {};


  /**
   * Tells the controller to halt loading of more dependencies.
   */
  goog.LoadController.prototype.pause = function() {};


  /**
   * Tells the controller to resume loading of more dependencies if paused.
   */
  goog.LoadController.prototype.resume = function() {};


  /**
   * Tells the controller that this dependency has finished loading.
   *
   * This causes this to be removed from pending() and any load callbacks to
   * fire.
   */
  goog.LoadController.prototype.loaded = function() {};


  /**
   * List of dependencies on which load has been called but which have not
   * called loaded on their controller. This includes the current dependency.
   *
   * @return {!Array<!goog.Dependency>}
   */
  goog.LoadController.prototype.pending = function() {};


  /**
   * Registers an object as an ES6 module's exports so that goog.modules may
   * require it by path.
   *
   * @param {string} path Full path of the module.
   * @param {?} exports
   * @param {string=} opt_closureNamespace Closure namespace to associate with
   *     this module.
   */
  goog.LoadController.prototype.registerEs6ModuleExports = function(
      path, exports, opt_closureNamespace) {};


  /**
   * Sets the current module state.
   *
   * @param {goog.ModuleType} type Type of module.
   */
  goog.LoadController.prototype.setModuleState = function(type) {};


  /**
   * Clears the current module state.
   */
  goog.LoadController.prototype.clearModuleState = function() {};


  /**
   * Registers a callback to call once the dependency is actually requested
   * via goog.require + all of the immediate dependencies have been loaded or
   * all other files have been loaded. Allows for lazy loading until
   * require'd without pausing dependency loading, which is needed on old IE.
   *
   * @param {!Function} callback
   */
  goog.LoadController.prototype.defer = function(callback) {};


  /**
   * @return {boolean}
   */
  goog.LoadController.prototype.areDepsLoaded = function() {};


  /**
   * Basic super class for all dependencies Closure Library can load.
   *
   * This default implementation is designed to load untranspiled, non-module
   * scripts in a web broswer.
   *
   * For transpiled non-goog.module files {@see goog.TranspiledDependency}.
   * For goog.modules see {@see goog.GoogModuleDependency}.
   * For untranspiled ES6 modules {@see goog.Es6ModuleDependency}.
   *
   * @param {string} path Absolute path of this script.
   * @param {string} relativePath Path of this script relative to goog.basePath.
   * @param {!Array<string>} provides goog.provided or goog.module symbols
   *     in this file.
   * @param {!Array<string>} requires goog symbols or relative paths to Closure
   *     this depends on.
   * @param {!Object<string, string>} loadFlags
   * @struct @constructor
   */
  goog.Dependency = function(
      path, relativePath, provides, requires, loadFlags) {
    /** @const */
    this.path = path;
    /** @const */
    this.relativePath = relativePath;
    /** @const */
    this.provides = provides;
    /** @const */
    this.requires = requires;
    /** @const */
    this.loadFlags = loadFlags;
    /** @private {boolean} */
    this.loaded_ = false;
    /** @private {!Array<function()>} */
    this.loadCallbacks_ = [];
  };


  /**
   * @return {string} The pathname part of this dependency's path if it is a
   *     URI.
   */
  goog.Dependency.prototype.getPathName = function() {
    var pathName = this.path;
    var protocolIndex = pathName.indexOf('://');
    if (protocolIndex >= 0) {
      pathName = pathName.substring(protocolIndex + 3);
      var slashIndex = pathName.indexOf('/');
      if (slashIndex >= 0) {
        pathName = pathName.substring(slashIndex + 1);
      }
    }
    return pathName;
  };


  /**
   * @param {function()} callback Callback to fire as soon as this has loaded.
   * @final
   */
  goog.Dependency.prototype.onLoad = function(callback) {
    if (this.loaded_) {
      callback();
    } else {
      this.loadCallbacks_.push(callback);
    }
  };


  /**
   * Marks this dependency as loaded and fires any callbacks registered with
   * onLoad.
   * @final
   */
  goog.Dependency.prototype.loaded = function() {
    this.loaded_ = true;
    var callbacks = this.loadCallbacks_;
    this.loadCallbacks_ = [];
    for (var i = 0; i < callbacks.length; i++) {
      callbacks[i]();
    }
  };


  /**
   * Whether or not document.written / appended script tags should be deferred.
   *
   * @private {boolean}
   */
  goog.Dependency.defer_ = false;


  /**
   * Map of script ready / state change callbacks. Old IE cannot handle putting
   * these properties on goog.global.
   *
   * @private @const {!Object<string, function(?):undefined>}
   */
  goog.Dependency.callbackMap_ = {};


  /**
   * @param {function(...?):?} callback
   * @return {string}
   * @private
   */
  goog.Dependency.registerCallback_ = function(callback) {
    var key = Math.random().toString(32);
    goog.Dependency.callbackMap_[key] = callback;
    return key;
  };


  /**
   * @param {string} key
   * @private
   */
  goog.Dependency.unregisterCallback_ = function(key) {
    delete goog.Dependency.callbackMap_[key];
  };


  /**
   * @param {string} key
   * @param {...?} var_args
   * @private
   * @suppress {unusedPrivateMembers}
   */
  goog.Dependency.callback_ = function(key, var_args) {
    if (key in goog.Dependency.callbackMap_) {
      var callback = goog.Dependency.callbackMap_[key];
      var args = [];
      for (var i = 1; i < arguments.length; i++) {
        args.push(arguments[i]);
      }
      callback.apply(undefined, args);
    } else {
      var errorMessage = 'Callback key ' + key +
          ' does not exist (was base.js loaded more than once?).';
      // MOE:begin_strip
      // TODO(johnplaisted): Some people internally are mistakenly loading
      // base.js twice, and this can happen while a dependency is loading,
      // wiping out state.
      goog.logToConsole_(errorMessage);
      // MOE:end_strip
      // MOE:insert throw Error(errorMessage);
    }
  };


  /**
   * Starts loading this dependency. This dependency can pause loading if it
   * needs to and resume it later via the controller interface.
   *
   * When this is loaded it should call controller.loaded(). Note that this will
   * end up calling the loaded method of this dependency; there is no need to
   * call it explicitly.
   *
   * @param {!goog.LoadController} controller
   */
  goog.Dependency.prototype.load = function(controller) {
    if (goog.global.CLOSURE_IMPORT_SCRIPT) {
      if (goog.global.CLOSURE_IMPORT_SCRIPT(this.path)) {
        controller.loaded();
      } else {
        controller.pause();
      }
      return;
    }

    if (!goog.inHtmlDocument_()) {
      goog.logToConsole_(
          'Cannot use default debug loader outside of HTML documents.');
      if (this.relativePath == 'deps.js') {
        // Some old code is relying on base.js auto loading deps.js failing with
        // no error before later setting CLOSURE_IMPORT_SCRIPT.
        // CLOSURE_IMPORT_SCRIPT should be set *before* base.js is loaded, or
        // CLOSURE_NO_DEPS set to true.
        goog.logToConsole_(
            'Consider setting CLOSURE_IMPORT_SCRIPT before loading base.js, ' +
            'or setting CLOSURE_NO_DEPS to true.');
        controller.loaded();
      } else {
        controller.pause();
      }
      return;
    }

    /** @type {!HTMLDocument} */
    var doc = goog.global.document;

    // If the user tries to require a new symbol after document load,
    // something has gone terribly wrong. Doing a document.write would
    // wipe out the page. This does not apply to the CSP-compliant method
    // of writing script tags.
    if (doc.readyState == 'complete' &&
        !goog.ENABLE_CHROME_APP_SAFE_SCRIPT_LOADING) {
      // Certain test frameworks load base.js multiple times, which tries
      // to write deps.js each time. If that happens, just fail silently.
      // These frameworks wipe the page between each load of base.js, so this
      // is OK.
      var isDeps = /\bdeps.js$/.test(this.path);
      if (isDeps) {
        controller.loaded();
        return;
      } else {
        throw Error('Cannot write "' + this.path + '" after document load');
      }
    }

    if (!goog.ENABLE_CHROME_APP_SAFE_SCRIPT_LOADING &&
        goog.isDocumentLoading_()) {
      var key = goog.Dependency.registerCallback_(function(script) {
        if (!goog.DebugLoader_.IS_OLD_IE_ || script.readyState == 'complete') {
          goog.Dependency.unregisterCallback_(key);
          controller.loaded();
        }
      });
      var nonceAttr = !goog.DebugLoader_.IS_OLD_IE_ && goog.getScriptNonce() ?
          ' nonce="' + goog.getScriptNonce() + '"' :
          '';
      var event =
          goog.DebugLoader_.IS_OLD_IE_ ? 'onreadystatechange' : 'onload';
      var defer = goog.Dependency.defer_ ? 'defer' : '';
      var script = '<script src="' + this.path + '" ' + event +
          '="goog.Dependency.callback_(\'' + key +
          '\', this)" type="text/javascript" ' + defer + nonceAttr + '><' +
          '/script>';
      doc.write(
          goog.TRUSTED_TYPES_POLICY_ ?
              goog.TRUSTED_TYPES_POLICY_.createHTML(script) :
              script);
    } else {
      var scriptEl =
          /** @type {!HTMLScriptElement} */ (doc.createElement('script'));
      scriptEl.defer = goog.Dependency.defer_;
      scriptEl.async = false;
      scriptEl.type = 'text/javascript';

      // If CSP nonces are used, propagate them to dynamically created scripts.
      // This is necessary to allow nonce-based CSPs without 'strict-dynamic'.
      var nonce = goog.getScriptNonce();
      if (nonce) {
        scriptEl.setAttribute('nonce', nonce);
      }

      if (goog.DebugLoader_.IS_OLD_IE_) {
        // Execution order is not guaranteed on old IE, halt loading and write
        // these scripts one at a time, after each loads.
        controller.pause();
        scriptEl.onreadystatechange = function() {
          if (scriptEl.readyState == 'loaded' ||
              scriptEl.readyState == 'complete') {
            controller.loaded();
            controller.resume();
          }
        };
      } else {
        scriptEl.onload = function() {
          scriptEl.onload = null;
          controller.loaded();
        };
      }

      scriptEl.src = goog.TRUSTED_TYPES_POLICY_ ?
          goog.TRUSTED_TYPES_POLICY_.createScriptURL(this.path) :
          this.path;
      doc.head.appendChild(scriptEl);
    }
  };


  /**
   * @param {string} path Absolute path of this script.
   * @param {string} relativePath Path of this script relative to goog.basePath.
   * @param {!Array<string>} provides Should be an empty array.
   *     TODO(johnplaisted) add support for adding closure namespaces to ES6
   *     modules for interop purposes.
   * @param {!Array<string>} requires goog symbols or relative paths to Closure
   *     this depends on.
   * @param {!Object<string, string>} loadFlags
   * @struct @constructor
   * @extends {goog.Dependency}
   */
  goog.Es6ModuleDependency = function(
      path, relativePath, provides, requires, loadFlags) {
    goog.Es6ModuleDependency.base(
        this, 'constructor', path, relativePath, provides, requires, loadFlags);
  };
  goog.inherits(goog.Es6ModuleDependency, goog.Dependency);


  /** @override */
  goog.Es6ModuleDependency.prototype.load = function(controller) {
    if (goog.global.CLOSURE_IMPORT_SCRIPT) {
      if (goog.global.CLOSURE_IMPORT_SCRIPT(this.path)) {
        controller.loaded();
      } else {
        controller.pause();
      }
      return;
    }

    if (!goog.inHtmlDocument_()) {
      goog.logToConsole_(
          'Cannot use default debug loader outside of HTML documents.');
      controller.pause();
      return;
    }

    /** @type {!HTMLDocument} */
    var doc = goog.global.document;

    var dep = this;

    // TODO(johnplaisted): Does document.writing really speed up anything? Any
    // difference between this and just waiting for interactive mode and then
    // appending?
    function write(src, contents) {
      if (contents) {
        var script = '<script type="module" crossorigin>' + contents + '</' +
            'script>';
        doc.write(
            goog.TRUSTED_TYPES_POLICY_ ?
                goog.TRUSTED_TYPES_POLICY_.createHTML(script) :
                script);
      } else {
        var script = '<script type="module" crossorigin src="' + src + '"></' +
            'script>';
        doc.write(
            goog.TRUSTED_TYPES_POLICY_ ?
                goog.TRUSTED_TYPES_POLICY_.createHTML(script) :
                script);
      }
    }

    function append(src, contents) {
      var scriptEl =
          /** @type {!HTMLScriptElement} */ (doc.createElement('script'));
      scriptEl.defer = true;
      scriptEl.async = false;
      scriptEl.type = 'module';
      scriptEl.setAttribute('crossorigin', true);

      // If CSP nonces are used, propagate them to dynamically created scripts.
      // This is necessary to allow nonce-based CSPs without 'strict-dynamic'.
      var nonce = goog.getScriptNonce();
      if (nonce) {
        scriptEl.setAttribute('nonce', nonce);
      }

      if (contents) {
        scriptEl.textContent = goog.TRUSTED_TYPES_POLICY_ ?
            goog.TRUSTED_TYPES_POLICY_.createScript(contents) :
            contents;
      } else {
        scriptEl.src = goog.TRUSTED_TYPES_POLICY_ ?
            goog.TRUSTED_TYPES_POLICY_.createScriptURL(src) :
            src;
      }

      doc.head.appendChild(scriptEl);
    }

    var create;

    if (goog.isDocumentLoading_()) {
      create = write;
      // We can ONLY call document.write if we are guaranteed that any
      // non-module script tags document.written after this are deferred.
      // Small optimization, in theory document.writing is faster.
      goog.Dependency.defer_ = true;
    } else {
      create = append;
    }

    // Write 4 separate tags here:
    // 1) Sets the module state at the correct time (just before execution).
    // 2) A src node for this, which just hopefully lets the browser load it a
    //    little early (no need to parse #3).
    // 3) Import the module and register it.
    // 4) Clear the module state at the correct time. Guaranteed to run even
    //    if there is an error in the module (#3 will not run if there is an
    //    error in the module).
    var beforeKey = goog.Dependency.registerCallback_(function() {
      goog.Dependency.unregisterCallback_(beforeKey);
      controller.setModuleState(goog.ModuleType.ES6);
    });
    create(undefined, 'goog.Dependency.callback_("' + beforeKey + '")');

    // TODO(johnplaisted): Does this really speed up anything?
    create(this.path, undefined);

    var registerKey = goog.Dependency.registerCallback_(function(exports) {
      goog.Dependency.unregisterCallback_(registerKey);
      controller.registerEs6ModuleExports(
          dep.path, exports, goog.moduleLoaderState_.moduleName);
    });
    create(
        undefined,
        'import * as m from "' + this.path + '"; goog.Dependency.callback_("' +
            registerKey + '", m)');

    var afterKey = goog.Dependency.registerCallback_(function() {
      goog.Dependency.unregisterCallback_(afterKey);
      controller.clearModuleState();
      controller.loaded();
    });
    create(undefined, 'goog.Dependency.callback_("' + afterKey + '")');
  };


  /**
   * Superclass of any dependency that needs to be loaded into memory,
   * transformed, and then eval'd (goog.modules and transpiled files).
   *
   * @param {string} path Absolute path of this script.
   * @param {string} relativePath Path of this script relative to goog.basePath.
   * @param {!Array<string>} provides goog.provided or goog.module symbols
   *     in this file.
   * @param {!Array<string>} requires goog symbols or relative paths to Closure
   *     this depends on.
   * @param {!Object<string, string>} loadFlags
   * @struct @constructor @abstract
   * @extends {goog.Dependency}
   */
  goog.TransformedDependency = function(
      path, relativePath, provides, requires, loadFlags) {
    goog.TransformedDependency.base(
        this, 'constructor', path, relativePath, provides, requires, loadFlags);
    /** @private {?string} */
    this.contents_ = null;

    /**
     * Whether to lazily make the synchronous XHR (when goog.require'd) or make
     * the synchronous XHR when initially loading. On FireFox 61 there is a bug
     * where an ES6 module cannot make a synchronous XHR (rather, it can, but if
     * it does then no other ES6 modules will load after).
     *
     * tl;dr we lazy load due to bugs on older browsers and eager load due to
     * bugs on newer ones.
     *
     * https://bugzilla.mozilla.org/show_bug.cgi?id=1477090
     *
     * @private @const {boolean}
     */
    this.lazyFetch_ = !goog.inHtmlDocument_() ||
        !('noModule' in goog.global.document.createElement('script'));
  };
  goog.inherits(goog.TransformedDependency, goog.Dependency);


  /** @override */
  goog.TransformedDependency.prototype.load = function(controller) {
    var dep = this;

    function fetch() {
      dep.contents_ = goog.loadFileSync_(dep.path);

      if (dep.contents_) {
        dep.contents_ = dep.transform(dep.contents_);
        if (dep.contents_) {
          dep.contents_ += '\n//# sourceURL=' + dep.path;
        }
      }
    }

    if (goog.global.CLOSURE_IMPORT_SCRIPT) {
      fetch();
      if (this.contents_ &&
          goog.global.CLOSURE_IMPORT_SCRIPT('', this.contents_)) {
        this.contents_ = null;
        controller.loaded();
      } else {
        controller.pause();
      }
      return;
    }


    var isEs6 = this.loadFlags['module'] == goog.ModuleType.ES6;

    if (!this.lazyFetch_) {
      fetch();
    }

    function load() {
      if (dep.lazyFetch_) {
        fetch();
      }

      if (!dep.contents_) {
        // loadFileSync_ or transform are responsible. Assume they logged an
        // error.
        return;
      }

      if (isEs6) {
        controller.setModuleState(goog.ModuleType.ES6);
      }

      var namespace;

      try {
        var contents = dep.contents_;
        dep.contents_ = null;
        goog.globalEval(contents);
        if (isEs6) {
          namespace = goog.moduleLoaderState_.moduleName;
        }
      } finally {
        if (isEs6) {
          controller.clearModuleState();
        }
      }

      if (isEs6) {
        // Due to circular dependencies this may not be available for require
        // right now.
        goog.global['$jscomp']['require']['ensure'](
            [dep.getPathName()], function() {
              controller.registerEs6ModuleExports(
                  dep.path,
                  goog.global['$jscomp']['require'](dep.getPathName()),
                  namespace);
            });
      }

      controller.loaded();
    }

    // Do not fetch now; in FireFox 47 the synchronous XHR doesn't block all
    // events. If we fetched now and then document.write'd the contents the
    // document.write would be an eval and would execute too soon! Instead write
    // a script tag to fetch and eval synchronously at the correct time.
    function fetchInOwnScriptThenLoad() {
      /** @type {!HTMLDocument} */
      var doc = goog.global.document;

      var key = goog.Dependency.registerCallback_(function() {
        goog.Dependency.unregisterCallback_(key);
        load();
      });

      var script = '<script type="text/javascript">' +
          goog.protectScriptTag_('goog.Dependency.callback_("' + key + '");') +
          '</' +
          'script>';
      doc.write(
          goog.TRUSTED_TYPES_POLICY_ ?
              goog.TRUSTED_TYPES_POLICY_.createHTML(script) :
              script);
    }

    // If one thing is pending it is this.
    var anythingElsePending = controller.pending().length > 1;

    // If anything else is loading we need to lazy load due to bugs in old IE.
    // Specifically script tags with src and script tags with contents could
    // execute out of order if document.write is used, so we cannot use
    // document.write. Do not pause here; it breaks old IE as well.
    var useOldIeWorkAround =
        anythingElsePending && goog.DebugLoader_.IS_OLD_IE_;

    // Additionally if we are meant to defer scripts but the page is still
    // loading (e.g. an ES6 module is loading) then also defer. Or if we are
    // meant to defer and anything else is pending then defer (those may be
    // scripts that did not need transformation and are just script tags with
    // defer set to true, and we need to evaluate after that deferred script).
    var needsAsyncLoading = goog.Dependency.defer_ &&
        (anythingElsePending || goog.isDocumentLoading_());

    if (useOldIeWorkAround || needsAsyncLoading) {
      // Note that we only defer when we have to rather than 100% of the time.
      // Always defering would work, but then in theory the order of
      // goog.require calls would then matter. We want to enforce that most of
      // the time the order of the require calls does not matter.
      controller.defer(function() {
        load();
      });
      return;
    }
    // TODO(johnplaisted): Externs are missing onreadystatechange for
    // HTMLDocument.
    /** @type {?} */
    var doc = goog.global.document;

    var isInternetExplorer =
        goog.inHtmlDocument_() && 'ActiveXObject' in goog.global;

    // Don't delay in any version of IE. There's bug around this that will
    // cause out of order script execution. This means that on older IE ES6
    // modules will load too early (while the document is still loading + the
    // dom is not available). The other option is to load too late (when the
    // document is complete and the onload even will never fire). This seems
    // to be the lesser of two evils as scripts already act like the former.
    if (isEs6 && goog.inHtmlDocument_() && goog.isDocumentLoading_() &&
        !isInternetExplorer) {
      goog.Dependency.defer_ = true;
      // Transpiled ES6 modules still need to load like regular ES6 modules,
      // aka only after the document is interactive.
      controller.pause();
      var oldCallback = doc.onreadystatechange;
      doc.onreadystatechange = function() {
        if (doc.readyState == 'interactive') {
          doc.onreadystatechange = oldCallback;
          load();
          controller.resume();
        }
        if (goog.isFunction(oldCallback)) {
          oldCallback.apply(undefined, arguments);
        }
      };
    } else {
      // Always eval on old IE.
      if (goog.DebugLoader_.IS_OLD_IE_ || !goog.inHtmlDocument_() ||
          !goog.isDocumentLoading_()) {
        load();
      } else {
        fetchInOwnScriptThenLoad();
      }
    }
  };


  /**
   * @param {string} contents
   * @return {string}
   * @abstract
   */
  goog.TransformedDependency.prototype.transform = function(contents) {};


  /**
   * Any non-goog.module dependency which needs to be transpiled before eval.
   *
   * @param {string} path Absolute path of this script.
   * @param {string} relativePath Path of this script relative to goog.basePath.
   * @param {!Array<string>} provides goog.provided or goog.module symbols
   *     in this file.
   * @param {!Array<string>} requires goog symbols or relative paths to Closure
   *     this depends on.
   * @param {!Object<string, string>} loadFlags
   * @param {!goog.Transpiler} transpiler
   * @struct @constructor
   * @extends {goog.TransformedDependency}
   */
  goog.TranspiledDependency = function(
      path, relativePath, provides, requires, loadFlags, transpiler) {
    goog.TranspiledDependency.base(
        this, 'constructor', path, relativePath, provides, requires, loadFlags);
    /** @protected @const*/
    this.transpiler = transpiler;
  };
  goog.inherits(goog.TranspiledDependency, goog.TransformedDependency);


  /** @override */
  goog.TranspiledDependency.prototype.transform = function(contents) {
    // Transpile with the pathname so that ES6 modules are domain agnostic.
    return this.transpiler.transpile(contents, this.getPathName());
  };


  /**
   * An ES6 module dependency that was transpiled to a jscomp module outside
   * of the debug loader, e.g. server side.
   *
   * @param {string} path Absolute path of this script.
   * @param {string} relativePath Path of this script relative to goog.basePath.
   * @param {!Array<string>} provides goog.provided or goog.module symbols
   *     in this file.
   * @param {!Array<string>} requires goog symbols or relative paths to Closure
   *     this depends on.
   * @param {!Object<string, string>} loadFlags
   * @struct @constructor
   * @extends {goog.TransformedDependency}
   */
  goog.PreTranspiledEs6ModuleDependency = function(
      path, relativePath, provides, requires, loadFlags) {
    goog.PreTranspiledEs6ModuleDependency.base(
        this, 'constructor', path, relativePath, provides, requires, loadFlags);
  };
  goog.inherits(
      goog.PreTranspiledEs6ModuleDependency, goog.TransformedDependency);


  /** @override */
  goog.PreTranspiledEs6ModuleDependency.prototype.transform = function(
      contents) {
    return contents;
  };


  /**
   * A goog.module, transpiled or not. Will always perform some minimal
   * transformation even when not transpiled to wrap in a goog.loadModule
   * statement.
   *
   * @param {string} path Absolute path of this script.
   * @param {string} relativePath Path of this script relative to goog.basePath.
   * @param {!Array<string>} provides goog.provided or goog.module symbols
   *     in this file.
   * @param {!Array<string>} requires goog symbols or relative paths to Closure
   *     this depends on.
   * @param {!Object<string, string>} loadFlags
   * @param {boolean} needsTranspile
   * @param {!goog.Transpiler} transpiler
   * @struct @constructor
   * @extends {goog.TransformedDependency}
   */
  goog.GoogModuleDependency = function(
      path, relativePath, provides, requires, loadFlags, needsTranspile,
      transpiler) {
    goog.GoogModuleDependency.base(
        this, 'constructor', path, relativePath, provides, requires, loadFlags);
    /** @private @const */
    this.needsTranspile_ = needsTranspile;
    /** @private @const */
    this.transpiler_ = transpiler;
  };
  goog.inherits(goog.GoogModuleDependency, goog.TransformedDependency);


  /** @override */
  goog.GoogModuleDependency.prototype.transform = function(contents) {
    if (this.needsTranspile_) {
      contents = this.transpiler_.transpile(contents, this.getPathName());
    }

    if (!goog.LOAD_MODULE_USING_EVAL || goog.global.JSON === undefined) {
      return '' +
          'goog.loadModule(function(exports) {' +
          '"use strict";' + contents +
          '\n' +  // terminate any trailing single line comment.
          ';return exports' +
          '});' +
          '\n//# sourceURL=' + this.path + '\n';
    } else {
      return '' +
          'goog.loadModule(' +
          goog.global.JSON.stringify(
              contents + '\n//# sourceURL=' + this.path + '\n') +
          ');';
    }
  };


  /**
   * Whether the browser is IE9 or earlier, which needs special handling
   * for deferred modules.
   * @const @private {boolean}
   */
  goog.DebugLoader_.IS_OLD_IE_ = !!(
      !goog.global.atob && goog.global.document && goog.global.document['all']);


  /**
   * @param {string} relPath
   * @param {!Array<string>|undefined} provides
   * @param {!Array<string>} requires
   * @param {boolean|!Object<string>=} opt_loadFlags
   * @see goog.addDependency
   */
  goog.DebugLoader_.prototype.addDependency = function(
      relPath, provides, requires, opt_loadFlags) {
    provides = provides || [];
    relPath = relPath.replace(/\\/g, '/');
    var path = goog.normalizePath_(goog.basePath + relPath);
    if (!opt_loadFlags || typeof opt_loadFlags === 'boolean') {
      opt_loadFlags = opt_loadFlags ? {'module': goog.ModuleType.GOOG} : {};
    }
    var dep = this.factory_.createDependency(
        path, relPath, provides, requires, opt_loadFlags,
        goog.transpiler_.needsTranspile(
            opt_loadFlags['lang'] || 'es3', opt_loadFlags['module']));
    this.dependencies_[path] = dep;
    for (var i = 0; i < provides.length; i++) {
      this.idToPath_[provides[i]] = path;
    }
    this.idToPath_[relPath] = path;
  };


  /**
   * Creates goog.Dependency instances for the debug loader to load.
   *
   * Should be overridden to have the debug loader use custom subclasses of
   * goog.Dependency.
   *
   * @param {!goog.Transpiler} transpiler
   * @struct @constructor
   */
  goog.DependencyFactory = function(transpiler) {
    /** @protected @const */
    this.transpiler = transpiler;
  };


  /**
   * @param {string} path Absolute path of the file.
   * @param {string} relativePath Path relative to closure’s base.js.
   * @param {!Array<string>} provides Array of provided goog.provide/module ids.
   * @param {!Array<string>} requires Array of required goog.provide/module /
   *     relative ES6 module paths.
   * @param {!Object<string, string>} loadFlags
   * @param {boolean} needsTranspile True if the file needs to be transpiled
   *     per the goog.Transpiler.
   * @return {!goog.Dependency}
   */
  goog.DependencyFactory.prototype.createDependency = function(
      path, relativePath, provides, requires, loadFlags, needsTranspile) {
    // MOE:begin_strip
    var provide, require;
    for (var i = 0; provide = provides[i]; i++) {
      goog.dependencies_.nameToPath[provide] = relativePath;
      goog.dependencies_.loadFlags[relativePath] = loadFlags;
    }
    for (var j = 0; require = requires[j]; j++) {
      if (!(relativePath in goog.dependencies_.requires)) {
        goog.dependencies_.requires[relativePath] = {};
      }
      goog.dependencies_.requires[relativePath][require] = true;
    }
    // MOE:end_strip

    if (loadFlags['module'] == goog.ModuleType.GOOG) {
      return new goog.GoogModuleDependency(
          path, relativePath, provides, requires, loadFlags, needsTranspile,
          this.transpiler);
    } else if (needsTranspile) {
      return new goog.TranspiledDependency(
          path, relativePath, provides, requires, loadFlags, this.transpiler);
    } else {
      if (loadFlags['module'] == goog.ModuleType.ES6) {
        if (goog.TRANSPILE == 'never' && goog.ASSUME_ES_MODULES_TRANSPILED) {
          return new goog.PreTranspiledEs6ModuleDependency(
              path, relativePath, provides, requires, loadFlags);
        } else {
          return new goog.Es6ModuleDependency(
              path, relativePath, provides, requires, loadFlags);
        }
      } else {
        return new goog.Dependency(
            path, relativePath, provides, requires, loadFlags);
      }
    }
  };


  /** @private @const */
  goog.debugLoader_ = new goog.DebugLoader_();


  /**
   * Loads the Closure Dependency file.
   *
   * Exposed a public function so CLOSURE_NO_DEPS can be set to false, base
   * loaded, setDependencyFactory called, and then this called. i.e. allows
   * custom loading of the deps file.
   */
  goog.loadClosureDeps = function() {
    goog.debugLoader_.loadClosureDeps();
  };


  /**
   * Sets the dependency factory, which can be used to create custom
   * goog.Dependency implementations to control how dependencies are loaded.
   *
   * Note: if you wish to call this function and provide your own implemnetation
   * it is a wise idea to set CLOSURE_NO_DEPS to true, otherwise the dependency
   * file and all of its goog.addDependency calls will use the default factory.
   * You can call goog.loadClosureDeps to load the Closure dependency file
   * later, after your factory is injected.
   *
   * @param {!goog.DependencyFactory} factory
   */
  goog.setDependencyFactory = function(factory) {
    goog.debugLoader_.setDependencyFactory(factory);
  };


  if (!goog.global.CLOSURE_NO_DEPS) {
    goog.debugLoader_.loadClosureDeps();
  }


  /**
   * Bootstraps the given namespaces and calls the callback once they are
   * available either via goog.require. This is a replacement for using
   * `goog.require` to bootstrap Closure JavaScript. Previously a `goog.require`
   * in an HTML file would guarantee that the require'd namespace was available
   * in the next immediate script tag. With ES6 modules this no longer a
   * guarantee.
   *
   * @param {!Array<string>} namespaces
   * @param {function(): ?} callback Function to call once all the namespaces
   *     have loaded. Always called asynchronously.
   */
  goog.bootstrap = function(namespaces, callback) {
    goog.debugLoader_.bootstrap(namespaces, callback);
  };
}


/**
 * @define {string} Trusted Types policy name. If non-empty then Closure will
 * use Trusted Types.
 */
goog.TRUSTED_TYPES_POLICY_NAME =
    goog.define('goog.TRUSTED_TYPES_POLICY_NAME', '');


/**
 * Returns the parameter.
 * @param {string} s
 * @return {string}
 * @private
 */
goog.identity_ = function(s) {
  return s;
};


/**
 * Creates Trusted Types policy if Trusted Types are supported by the browser.
 * The policy just blesses any string as a Trusted Type. It is not visibility
 * restricted because anyone can also call TrustedTypes.createPolicy directly.
 * However, the allowed names should be restricted by a HTTP header and the
 * reference to the created policy should be visibility restricted.
 * @param {string} name
 * @return {?TrustedTypePolicy}
 */
goog.createTrustedTypesPolicy = function(name) {
  var policy = null;
  // TODO(koto): Remove window.TrustedTypes variant when the newer API ships.
  var policyFactory = goog.global.trustedTypes || goog.global.TrustedTypes;
  if (!policyFactory || !policyFactory.createPolicy) {
    return policy;
  }
  // TrustedTypes.createPolicy throws if called with a name that is already
  // registered, even in report-only mode. Until the API changes, catch the
  // error not to break the applications functionally. In such case, the code
  // will fall back to using regular Safe Types.
  // TODO(koto): Remove catching once createPolicy API stops throwing.
  try {
    policy = policyFactory.createPolicy(name, {
      createHTML: goog.identity_,
      createScript: goog.identity_,
      createScriptURL: goog.identity_,
      createURL: goog.identity_
    });
  } catch (e) {
    goog.logToConsole_(e.message);
  }
  return policy;
};


/** @private @const {?TrustedTypePolicy} */
goog.TRUSTED_TYPES_POLICY_ = goog.TRUSTED_TYPES_POLICY_NAME ?
    goog.createTrustedTypesPolicy(goog.TRUSTED_TYPES_POLICY_NAME + '#base') :
    null;

//javascript/closure/transitionalforwarddeclarations.js
// Copyright 2017 The Closure Library Authors. All Rights Reserved.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//      http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS-IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

/**
 * @fileoverview The forward declarations in this file are here to faciliate
 * the removal of "deps.js" from the "base" rule.  These types are
 * included in various extern files.  These rules should be cleaned up
 * so that these declarations aren't necessary.
 * @suppress {extraRequire}
 */

goog.forwardDeclare('goog.Promise');
goog.forwardDeclare('goog.date.DateLike');
goog.forwardDeclare('goog.date.DateTime');
goog.forwardDeclare('goog.events.EventId');
goog.forwardDeclare('goog.events.Key');
goog.forwardDeclare('goog.events.KeyCodes');
goog.forwardDeclare('goog.i18n.TimeZone');
goog.forwardDeclare('goog.math.Range');
goog.forwardDeclare('goog.math.Size');
goog.forwardDeclare('goog.structs.Map');

//third_party/javascript/lodash/lodash.4.min.js
/**
 * @license
 * Lodash lodash.com/license | Underscore.js 1.8.3 underscorejs.org/LICENSE
 */
;
(/** @suppress {checkTypes|suspiciousCode|uselessCode|globalThis|checkVars} */
function(){function n(n,t,r){switch(r.length){case 0:return n.call(t);case 1:return n.call(t,r[0]);case 2:return n.call(t,r[0],r[1]);case 3:return n.call(t,r[0],r[1],r[2])}return n.apply(t,r)}function t(n,t,r,e){for(var u=-1,i=null==n?0:n.length;++u<i;){var o=n[u];t(e,o,r(o),n)}return e}function r(n,t){for(var r=-1,e=null==n?0:n.length;++r<e&&false!==t(n[r],r,n););return n}function e(n,t){for(var r=null==n?0:n.length;r--&&false!==t(n[r],r,n););return n}function u(n,t){for(var r=-1,e=null==n?0:n.length;++r<e;)if(!t(n[r],r,n))return false;
return true}function i(n,t){for(var r=-1,e=null==n?0:n.length,u=0,i=[];++r<e;){var o=n[r];t(o,r,n)&&(i[u++]=o)}return i}function o(n,t){return!(null==n||!n.length)&&-1<v(n,t,0)}function f(n,t,r){for(var e=-1,u=null==n?0:n.length;++e<u;)if(r(t,n[e]))return true;return false}function c(n,t){for(var r=-1,e=null==n?0:n.length,u=Array(e);++r<e;)u[r]=t(n[r],r,n);return u}function a(n,t){for(var r=-1,e=t.length,u=n.length;++r<e;)n[u+r]=t[r];return n}function l(n,t,r,e){var u=-1,i=null==n?0:n.length;for(e&&i&&(r=n[++u]);++u<i;)r=t(r,n[u],u,n);
return r}function s(n,t,r,e){var u=null==n?0:n.length;for(e&&u&&(r=n[--u]);u--;)r=t(r,n[u],u,n);return r}function h(n,t){for(var r=-1,e=null==n?0:n.length;++r<e;)if(t(n[r],r,n))return true;return false}function p(n,t,r){var e;return r(n,function(n,r,u){if(t(n,r,u))return e=r,false}),e}function _(n,t,r,e){var u=n.length;for(r+=e?1:-1;e?r--:++r<u;)if(t(n[r],r,n))return r;return-1}function v(n,t,r){if(t===t)n:{--r;for(var e=n.length;++r<e;)if(n[r]===t){n=r;break n}n=-1}else n=_(n,d,r);return n}function g(n,t,r,e){
--r;for(var u=n.length;++r<u;)if(e(n[r],t))return r;return-1}function d(n){return n!==n}function y(n,t){var r=null==n?0:n.length;return r?m(n,t)/r:F}function b(n){return function(t){return null==t?T:t[n]}}function x(n){return function(t){return null==n?T:n[t]}}function j(n,t,r,e,u){return u(n,function(n,u,i){r=e?(e=false,n):t(r,n,u,i)}),r}function w(n,t){var r=n.length;for(n.sort(t);r--;)n[r]=n[r].c;return n}function m(n,t){for(var r,e=-1,u=n.length;++e<u;){var i=t(n[e]);i!==T&&(r=r===T?i:r+i)}return r;
}function A(n,t){for(var r=-1,e=Array(n);++r<n;)e[r]=t(r);return e}function E(n,t){return c(t,function(t){return[t,n[t]]})}function k(n){return function(t){return n(t)}}function S(n,t){return c(t,function(t){return n[t]})}function O(n,t){return n.has(t)}function I(n,t){for(var r=-1,e=n.length;++r<e&&-1<v(t,n[r],0););return r}function R(n,t){for(var r=n.length;r--&&-1<v(t,n[r],0););return r}function z(n){return"\\"+Un[n]}function W(n){var t=-1,r=Array(n.size);return n.forEach(function(n,e){r[++t]=[e,n];
}),r}function B(n,t){return function(r){return n(t(r))}}function L(n,t){for(var r=-1,e=n.length,u=0,i=[];++r<e;){var o=n[r];o!==t&&"__lodash_placeholder__"!==o||(n[r]="__lodash_placeholder__",i[u++]=r)}return i}function U(n){var t=-1,r=Array(n.size);return n.forEach(function(n){r[++t]=n}),r}function C(n){var t=-1,r=Array(n.size);return n.forEach(function(n){r[++t]=[n,n]}),r}function D(n){if(Rn.test(n)){for(var t=On.lastIndex=0;On.test(n);)++t;n=t}else n=Qn(n);return n}function M(n){return Rn.test(n)?n.match(On)||[]:n.split("");
}var T,$=1/0,F=NaN,N=[["ary",128],["bind",1],["bindKey",2],["curry",8],["curryRight",16],["flip",512],["partial",32],["partialRight",64],["rearg",256]],P=/\b__p\+='';/g,Z=/\b(__p\+=)''\+/g,q=/(__e\(.*?\)|\b__t\))\+'';/g,V=/&(?:amp|lt|gt|quot|#39);/g,K=/[&<>"']/g,G=RegExp(V.source),H=RegExp(K.source),J=/<%-([\s\S]+?)%>/g,Y=/<%([\s\S]+?)%>/g,Q=/<%=([\s\S]+?)%>/g,X=/\.|\[(?:[^[\]]*|(["'])(?:(?!\1)[^\\]|\\.)*?\1)\]/,nn=/^\w*$/,tn=/[^.[\]]+|\[(?:(-?\d+(?:\.\d+)?)|(["'])((?:(?!\2)[^\\]|\\.)*?)\2)\]|(?=(?:\.|\[\])(?:\.|\[\]|$))/g,rn=/[\\^$.*+?()[\]{}|]/g,en=RegExp(rn.source),un=/^\s+|\s+$/g,on=/^\s+/,fn=/\s+$/,cn=/\{(?:\n\/\* \[wrapped with .+\] \*\/)?\n?/,an=/\{\n\/\* \[wrapped with (.+)\] \*/,ln=/,? & /,sn=/[^\x00-\x2f\x3a-\x40\x5b-\x60\x7b-\x7f]+/g,hn=/\\(\\)?/g,pn=/\$\{([^\\}]*(?:\\.[^\\}]*)*)\}/g,_n=/\w*$/,vn=/^[-+]0x[0-9a-f]+$/i,gn=/^0b[01]+$/i,dn=/^\[object .+?Constructor\]$/,yn=/^0o[0-7]+$/i,bn=/^(?:0|[1-9]\d*)$/,xn=/[\xc0-\xd6\xd8-\xf6\xf8-\xff\u0100-\u017f]/g,jn=/($^)/,wn=/['\n\r\u2028\u2029\\]/g,mn="[\\ufe0e\\ufe0f]?(?:[\\u0300-\\u036f\\ufe20-\\ufe2f\\u20d0-\\u20ff]|\\ud83c[\\udffb-\\udfff])?(?:\\u200d(?:[^\\ud800-\\udfff]|(?:\\ud83c[\\udde6-\\uddff]){2}|[\\ud800-\\udbff][\\udc00-\\udfff])[\\ufe0e\\ufe0f]?(?:[\\u0300-\\u036f\\ufe20-\\ufe2f\\u20d0-\\u20ff]|\\ud83c[\\udffb-\\udfff])?)*",An="(?:[\\u2700-\\u27bf]|(?:\\ud83c[\\udde6-\\uddff]){2}|[\\ud800-\\udbff][\\udc00-\\udfff])"+mn,En="(?:[^\\ud800-\\udfff][\\u0300-\\u036f\\ufe20-\\ufe2f\\u20d0-\\u20ff]?|[\\u0300-\\u036f\\ufe20-\\ufe2f\\u20d0-\\u20ff]|(?:\\ud83c[\\udde6-\\uddff]){2}|[\\ud800-\\udbff][\\udc00-\\udfff]|[\\ud800-\\udfff])",kn=RegExp("['\u2019]","g"),Sn=RegExp("[\\u0300-\\u036f\\ufe20-\\ufe2f\\u20d0-\\u20ff]","g"),On=RegExp("\\ud83c[\\udffb-\\udfff](?=\\ud83c[\\udffb-\\udfff])|"+En+mn,"g"),In=RegExp(["[A-Z\\xc0-\\xd6\\xd8-\\xde]?[a-z\\xdf-\\xf6\\xf8-\\xff]+(?:['\u2019](?:d|ll|m|re|s|t|ve))?(?=[\\xac\\xb1\\xd7\\xf7\\x00-\\x2f\\x3a-\\x40\\x5b-\\x60\\x7b-\\xbf\\u2000-\\u206f \\t\\x0b\\f\\xa0\\ufeff\\n\\r\\u2028\\u2029\\u1680\\u180e\\u2000\\u2001\\u2002\\u2003\\u2004\\u2005\\u2006\\u2007\\u2008\\u2009\\u200a\\u202f\\u205f\\u3000]|[A-Z\\xc0-\\xd6\\xd8-\\xde]|$)|(?:[A-Z\\xc0-\\xd6\\xd8-\\xde]|[^\\ud800-\\udfff\\xac\\xb1\\xd7\\xf7\\x00-\\x2f\\x3a-\\x40\\x5b-\\x60\\x7b-\\xbf\\u2000-\\u206f \\t\\x0b\\f\\xa0\\ufeff\\n\\r\\u2028\\u2029\\u1680\\u180e\\u2000\\u2001\\u2002\\u2003\\u2004\\u2005\\u2006\\u2007\\u2008\\u2009\\u200a\\u202f\\u205f\\u3000\\d+\\u2700-\\u27bfa-z\\xdf-\\xf6\\xf8-\\xffA-Z\\xc0-\\xd6\\xd8-\\xde])+(?:['\u2019](?:D|LL|M|RE|S|T|VE))?(?=[\\xac\\xb1\\xd7\\xf7\\x00-\\x2f\\x3a-\\x40\\x5b-\\x60\\x7b-\\xbf\\u2000-\\u206f \\t\\x0b\\f\\xa0\\ufeff\\n\\r\\u2028\\u2029\\u1680\\u180e\\u2000\\u2001\\u2002\\u2003\\u2004\\u2005\\u2006\\u2007\\u2008\\u2009\\u200a\\u202f\\u205f\\u3000]|[A-Z\\xc0-\\xd6\\xd8-\\xde](?:[a-z\\xdf-\\xf6\\xf8-\\xff]|[^\\ud800-\\udfff\\xac\\xb1\\xd7\\xf7\\x00-\\x2f\\x3a-\\x40\\x5b-\\x60\\x7b-\\xbf\\u2000-\\u206f \\t\\x0b\\f\\xa0\\ufeff\\n\\r\\u2028\\u2029\\u1680\\u180e\\u2000\\u2001\\u2002\\u2003\\u2004\\u2005\\u2006\\u2007\\u2008\\u2009\\u200a\\u202f\\u205f\\u3000\\d+\\u2700-\\u27bfa-z\\xdf-\\xf6\\xf8-\\xffA-Z\\xc0-\\xd6\\xd8-\\xde])|$)|[A-Z\\xc0-\\xd6\\xd8-\\xde]?(?:[a-z\\xdf-\\xf6\\xf8-\\xff]|[^\\ud800-\\udfff\\xac\\xb1\\xd7\\xf7\\x00-\\x2f\\x3a-\\x40\\x5b-\\x60\\x7b-\\xbf\\u2000-\\u206f \\t\\x0b\\f\\xa0\\ufeff\\n\\r\\u2028\\u2029\\u1680\\u180e\\u2000\\u2001\\u2002\\u2003\\u2004\\u2005\\u2006\\u2007\\u2008\\u2009\\u200a\\u202f\\u205f\\u3000\\d+\\u2700-\\u27bfa-z\\xdf-\\xf6\\xf8-\\xffA-Z\\xc0-\\xd6\\xd8-\\xde])+(?:['\u2019](?:d|ll|m|re|s|t|ve))?|[A-Z\\xc0-\\xd6\\xd8-\\xde]+(?:['\u2019](?:D|LL|M|RE|S|T|VE))?|\\d*(?:1ST|2ND|3RD|(?![123])\\dTH)(?=\\b|[a-z_])|\\d*(?:1st|2nd|3rd|(?![123])\\dth)(?=\\b|[A-Z_])|\\d+",An].join("|"),"g"),Rn=RegExp("[\\u200d\\ud800-\\udfff\\u0300-\\u036f\\ufe20-\\ufe2f\\u20d0-\\u20ff\\ufe0e\\ufe0f]"),zn=/[a-z][A-Z]|[A-Z]{2}[a-z]|[0-9][a-zA-Z]|[a-zA-Z][0-9]|[^a-zA-Z0-9 ]/,Wn="Array Buffer DataView Date Error Float32Array Float64Array Function Int8Array Int16Array Int32Array Map Math Object Promise RegExp Set String Symbol TypeError Uint8Array Uint8ClampedArray Uint16Array Uint32Array WeakMap _ clearTimeout isFinite parseInt setTimeout".split(" "),Bn={};
Bn["[object Float32Array]"]=Bn["[object Float64Array]"]=Bn["[object Int8Array]"]=Bn["[object Int16Array]"]=Bn["[object Int32Array]"]=Bn["[object Uint8Array]"]=Bn["[object Uint8ClampedArray]"]=Bn["[object Uint16Array]"]=Bn["[object Uint32Array]"]=true,Bn["[object Arguments]"]=Bn["[object Array]"]=Bn["[object ArrayBuffer]"]=Bn["[object Boolean]"]=Bn["[object DataView]"]=Bn["[object Date]"]=Bn["[object Error]"]=Bn["[object Function]"]=Bn["[object Map]"]=Bn["[object Number]"]=Bn["[object Object]"]=Bn["[object RegExp]"]=Bn["[object Set]"]=Bn["[object String]"]=Bn["[object WeakMap]"]=false;
var Ln={};Ln["[object Arguments]"]=Ln["[object Array]"]=Ln["[object ArrayBuffer]"]=Ln["[object DataView]"]=Ln["[object Boolean]"]=Ln["[object Date]"]=Ln["[object Float32Array]"]=Ln["[object Float64Array]"]=Ln["[object Int8Array]"]=Ln["[object Int16Array]"]=Ln["[object Int32Array]"]=Ln["[object Map]"]=Ln["[object Number]"]=Ln["[object Object]"]=Ln["[object RegExp]"]=Ln["[object Set]"]=Ln["[object String]"]=Ln["[object Symbol]"]=Ln["[object Uint8Array]"]=Ln["[object Uint8ClampedArray]"]=Ln["[object Uint16Array]"]=Ln["[object Uint32Array]"]=true,
Ln["[object Error]"]=Ln["[object Function]"]=Ln["[object WeakMap]"]=false;var Un={"\\":"\\","'":"'","\n":"n","\r":"r","\u2028":"u2028","\u2029":"u2029"},Cn=parseFloat,Dn=parseInt,Mn=typeof global=="object"&&global&&global.Object===Object&&global,Tn=typeof self=="object"&&self&&self.Object===Object&&self,$n=Mn||Tn||Function("return this")(),Fn=typeof exports=="object"&&exports&&!exports.nodeType&&exports,Nn=Fn&&typeof module=="object"&&module&&!module.nodeType&&module,Pn=Nn&&Nn.exports===Fn,Zn=Pn&&Mn.process,qn=function(){
try{var n=Nn&&Nn.f&&Nn.f("util").types;return n?n:Zn&&Zn.binding&&Zn.binding("util")}catch(n){}}(),Vn=qn&&qn.isArrayBuffer,Kn=qn&&qn.isDate,Gn=qn&&qn.isMap,Hn=qn&&qn.isRegExp,Jn=qn&&qn.isSet,Yn=qn&&qn.isTypedArray,Qn=b("length"),Xn=x({"\xc0":"A","\xc1":"A","\xc2":"A","\xc3":"A","\xc4":"A","\xc5":"A","\xe0":"a","\xe1":"a","\xe2":"a","\xe3":"a","\xe4":"a","\xe5":"a","\xc7":"C","\xe7":"c","\xd0":"D","\xf0":"d","\xc8":"E","\xc9":"E","\xca":"E","\xcb":"E","\xe8":"e","\xe9":"e","\xea":"e","\xeb":"e","\xcc":"I",
"\xcd":"I","\xce":"I","\xcf":"I","\xec":"i","\xed":"i","\xee":"i","\xef":"i","\xd1":"N","\xf1":"n","\xd2":"O","\xd3":"O","\xd4":"O","\xd5":"O","\xd6":"O","\xd8":"O","\xf2":"o","\xf3":"o","\xf4":"o","\xf5":"o","\xf6":"o","\xf8":"o","\xd9":"U","\xda":"U","\xdb":"U","\xdc":"U","\xf9":"u","\xfa":"u","\xfb":"u","\xfc":"u","\xdd":"Y","\xfd":"y","\xff":"y","\xc6":"Ae","\xe6":"ae","\xde":"Th","\xfe":"th","\xdf":"ss","\u0100":"A","\u0102":"A","\u0104":"A","\u0101":"a","\u0103":"a","\u0105":"a","\u0106":"C",
"\u0108":"C","\u010a":"C","\u010c":"C","\u0107":"c","\u0109":"c","\u010b":"c","\u010d":"c","\u010e":"D","\u0110":"D","\u010f":"d","\u0111":"d","\u0112":"E","\u0114":"E","\u0116":"E","\u0118":"E","\u011a":"E","\u0113":"e","\u0115":"e","\u0117":"e","\u0119":"e","\u011b":"e","\u011c":"G","\u011e":"G","\u0120":"G","\u0122":"G","\u011d":"g","\u011f":"g","\u0121":"g","\u0123":"g","\u0124":"H","\u0126":"H","\u0125":"h","\u0127":"h","\u0128":"I","\u012a":"I","\u012c":"I","\u012e":"I","\u0130":"I","\u0129":"i",
"\u012b":"i","\u012d":"i","\u012f":"i","\u0131":"i","\u0134":"J","\u0135":"j","\u0136":"K","\u0137":"k","\u0138":"k","\u0139":"L","\u013b":"L","\u013d":"L","\u013f":"L","\u0141":"L","\u013a":"l","\u013c":"l","\u013e":"l","\u0140":"l","\u0142":"l","\u0143":"N","\u0145":"N","\u0147":"N","\u014a":"N","\u0144":"n","\u0146":"n","\u0148":"n","\u014b":"n","\u014c":"O","\u014e":"O","\u0150":"O","\u014d":"o","\u014f":"o","\u0151":"o","\u0154":"R","\u0156":"R","\u0158":"R","\u0155":"r","\u0157":"r","\u0159":"r",
"\u015a":"S","\u015c":"S","\u015e":"S","\u0160":"S","\u015b":"s","\u015d":"s","\u015f":"s","\u0161":"s","\u0162":"T","\u0164":"T","\u0166":"T","\u0163":"t","\u0165":"t","\u0167":"t","\u0168":"U","\u016a":"U","\u016c":"U","\u016e":"U","\u0170":"U","\u0172":"U","\u0169":"u","\u016b":"u","\u016d":"u","\u016f":"u","\u0171":"u","\u0173":"u","\u0174":"W","\u0175":"w","\u0176":"Y","\u0177":"y","\u0178":"Y","\u0179":"Z","\u017b":"Z","\u017d":"Z","\u017a":"z","\u017c":"z","\u017e":"z","\u0132":"IJ","\u0133":"ij",
"\u0152":"Oe","\u0153":"oe","\u0149":"'n","\u017f":"s"}),nt=x({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}),tt=x({"&amp;":"&","&lt;":"<","&gt;":">","&quot;":'"',"&#39;":"'"}),rt=function x(mn){function An(n){if(yu(n)&&!ff(n)&&!(n instanceof Un)){if(n instanceof On)return n;if(oi.call(n,"__wrapped__"))return Fe(n)}return new On(n)}function En(){}function On(n,t){this.__wrapped__=n,this.__actions__=[],this.__chain__=!!t,this.__index__=0,this.__values__=T}function Un(n){this.__wrapped__=n,
this.__actions__=[],this.__dir__=1,this.__filtered__=false,this.__iteratees__=[],this.__takeCount__=4294967295,this.__views__=[]}function Mn(n){var t=-1,r=null==n?0:n.length;for(this.clear();++t<r;){var e=n[t];this.set(e[0],e[1])}}function Tn(n){var t=-1,r=null==n?0:n.length;for(this.clear();++t<r;){var e=n[t];this.set(e[0],e[1])}}function Fn(n){var t=-1,r=null==n?0:n.length;for(this.clear();++t<r;){var e=n[t];this.set(e[0],e[1])}}function Nn(n){var t=-1,r=null==n?0:n.length;for(this.__data__=new Fn;++t<r;)this.add(n[t]);
}function Zn(n){this.size=(this.__data__=new Tn(n)).size}function qn(n,t){var r,e=ff(n),u=!e&&of(n),i=!e&&!u&&af(n),o=!e&&!u&&!i&&_f(n),u=(e=e||u||i||o)?A(n.length,ni):[],f=u.length;for(r in n)!t&&!oi.call(n,r)||e&&("length"==r||i&&("offset"==r||"parent"==r)||o&&("buffer"==r||"byteLength"==r||"byteOffset"==r)||Se(r,f))||u.push(r);return u}function Qn(n){var t=n.length;return t?n[ir(0,t-1)]:T}function et(n,t){return De(Ur(n),pt(t,0,n.length))}function ut(n){return De(Ur(n))}function it(n,t,r){(r===T||lu(n[t],r))&&(r!==T||t in n)||st(n,t,r);
}function ot(n,t,r){var e=n[t];oi.call(n,t)&&lu(e,r)&&(r!==T||t in n)||st(n,t,r)}function ft(n,t){for(var r=n.length;r--;)if(lu(n[r][0],t))return r;return-1}function ct(n,t,r,e){return uo(n,function(n,u,i){t(e,n,r(n),i)}),e}function at(n,t){return n&&Cr(t,Wu(t),n)}function lt(n,t){return n&&Cr(t,Bu(t),n)}function st(n,t,r){"__proto__"==t&&Ai?Ai(n,t,{configurable:true,enumerable:true,value:r,writable:true}):n[t]=r}function ht(n,t){for(var r=-1,e=t.length,u=Ku(e),i=null==n;++r<e;)u[r]=i?T:Ru(n,t[r]);return u;
}function pt(n,t,r){return n===n&&(r!==T&&(n=n<=r?n:r),t!==T&&(n=n>=t?n:t)),n}function _t(n,t,e,u,i,o){var f,c=1&t,a=2&t,l=4&t;if(e&&(f=i?e(n,u,i,o):e(n)),f!==T)return f;if(!du(n))return n;if(u=ff(n)){if(f=me(n),!c)return Ur(n,f)}else{var s=vo(n),h="[object Function]"==s||"[object GeneratorFunction]"==s;if(af(n))return Ir(n,c);if("[object Object]"==s||"[object Arguments]"==s||h&&!i){if(f=a||h?{}:Ae(n),!c)return a?Mr(n,lt(f,n)):Dr(n,at(f,n))}else{if(!Ln[s])return i?n:{};f=Ee(n,s,c)}}if(o||(o=new Zn),
i=o.get(n))return i;o.set(n,f),pf(n)?n.forEach(function(r){f.add(_t(r,t,e,r,n,o))}):sf(n)&&n.forEach(function(r,u){f.set(u,_t(r,t,e,u,n,o))});var a=l?a?ve:_e:a?Bu:Wu,p=u?T:a(n);return r(p||n,function(r,u){p&&(u=r,r=n[u]),ot(f,u,_t(r,t,e,u,n,o))}),f}function vt(n){var t=Wu(n);return function(r){return gt(r,n,t)}}function gt(n,t,r){var e=r.length;if(null==n)return!e;for(n=Qu(n);e--;){var u=r[e],i=t[u],o=n[u];if(o===T&&!(u in n)||!i(o))return false}return true}function dt(n,t,r){if(typeof n!="function")throw new ti("Expected a function");
return bo(function(){n.apply(T,r)},t)}function yt(n,t,r,e){var u=-1,i=o,a=true,l=n.length,s=[],h=t.length;if(!l)return s;r&&(t=c(t,k(r))),e?(i=f,a=false):200<=t.length&&(i=O,a=false,t=new Nn(t));n:for(;++u<l;){var p=n[u],_=null==r?p:r(p),p=e||0!==p?p:0;if(a&&_===_){for(var v=h;v--;)if(t[v]===_)continue n;s.push(p)}else i(t,_,e)||s.push(p)}return s}function bt(n,t){var r=true;return uo(n,function(n,e,u){return r=!!t(n,e,u)}),r}function xt(n,t,r){for(var e=-1,u=n.length;++e<u;){var i=n[e],o=t(i);if(null!=o&&(f===T?o===o&&!wu(o):r(o,f)))var f=o,c=i;
}return c}function jt(n,t){var r=[];return uo(n,function(n,e,u){t(n,e,u)&&r.push(n)}),r}function wt(n,t,r,e,u){var i=-1,o=n.length;for(r||(r=ke),u||(u=[]);++i<o;){var f=n[i];0<t&&r(f)?1<t?wt(f,t-1,r,e,u):a(u,f):e||(u[u.length]=f)}return u}function mt(n,t){return n&&oo(n,t,Wu)}function At(n,t){return n&&fo(n,t,Wu)}function Et(n,t){return i(t,function(t){return _u(n[t])})}function kt(n,t){t=Sr(t,n);for(var r=0,e=t.length;null!=n&&r<e;)n=n[Me(t[r++])];return r&&r==e?n:T}function St(n,t,r){return t=t(n),
ff(n)?t:a(t,r(n))}function Ot(n){if(null==n)n=n===T?"[object Undefined]":"[object Null]";else if(mi&&mi in Qu(n)){var t=oi.call(n,mi),r=n[mi];try{n[mi]=T;var e=true}catch(n){}var u=ai.call(n);e&&(t?n[mi]=r:delete n[mi]),n=u}else n=ai.call(n);return n}function It(n,t){return n>t}function Rt(n,t){return null!=n&&oi.call(n,t)}function zt(n,t){return null!=n&&t in Qu(n)}function Wt(n,t,r){for(var e=r?f:o,u=n[0].length,i=n.length,a=i,l=Ku(i),s=1/0,h=[];a--;){var p=n[a];a&&t&&(p=c(p,k(t))),s=Ci(p.length,s),
l[a]=!r&&(t||120<=u&&120<=p.length)?new Nn(a&&p):T}var p=n[0],_=-1,v=l[0];n:for(;++_<u&&h.length<s;){var g=p[_],d=t?t(g):g,g=r||0!==g?g:0;if(v?!O(v,d):!e(h,d,r)){for(a=i;--a;){var y=l[a];if(y?!O(y,d):!e(n[a],d,r))continue n}v&&v.push(d),h.push(g)}}return h}function Bt(n,t,r){var e={};return mt(n,function(n,u,i){t(e,r(n),u,i)}),e}function Lt(t,r,e){return r=Sr(r,t),t=2>r.length?t:kt(t,hr(r,0,-1)),r=null==t?t:t[Me(Ve(r))],null==r?T:n(r,t,e)}function Ut(n){return yu(n)&&"[object Arguments]"==Ot(n)}function Ct(n){
return yu(n)&&"[object ArrayBuffer]"==Ot(n)}function Dt(n){return yu(n)&&"[object Date]"==Ot(n)}function Mt(n,t,r,e,u){if(n===t)t=true;else if(null==n||null==t||!yu(n)&&!yu(t))t=n!==n&&t!==t;else n:{var i=ff(n),o=ff(t),f=i?"[object Array]":vo(n),c=o?"[object Array]":vo(t),f="[object Arguments]"==f?"[object Object]":f,c="[object Arguments]"==c?"[object Object]":c,a="[object Object]"==f,o="[object Object]"==c;if((c=f==c)&&af(n)){if(!af(t)){t=false;break n}i=true,a=false}if(c&&!a)u||(u=new Zn),t=i||_f(n)?se(n,t,r,e,Mt,u):he(n,t,f,r,e,Mt,u);else{
if(!(1&r)&&(i=a&&oi.call(n,"__wrapped__"),f=o&&oi.call(t,"__wrapped__"),i||f)){n=i?n.value():n,t=f?t.value():t,u||(u=new Zn),t=Mt(n,t,r,e,u);break n}if(c)t:if(u||(u=new Zn),i=1&r,f=_e(n),o=f.length,c=_e(t).length,o==c||i){for(a=o;a--;){var l=f[a];if(!(i?l in t:oi.call(t,l))){t=false;break t}}if((c=u.get(n))&&u.get(t))t=c==t;else{c=true,u.set(n,t),u.set(t,n);for(var s=i;++a<o;){var l=f[a],h=n[l],p=t[l];if(e)var _=i?e(p,h,l,t,n,u):e(h,p,l,n,t,u);if(_===T?h!==p&&!Mt(h,p,r,e,u):!_){c=false;break}s||(s="constructor"==l);
}c&&!s&&(r=n.constructor,e=t.constructor,r!=e&&"constructor"in n&&"constructor"in t&&!(typeof r=="function"&&r instanceof r&&typeof e=="function"&&e instanceof e)&&(c=false)),u.delete(n),u.delete(t),t=c}}else t=false;else t=false}}return t}function Tt(n){return yu(n)&&"[object Map]"==vo(n)}function $t(n,t,r,e){var u=r.length,i=u,o=!e;if(null==n)return!i;for(n=Qu(n);u--;){var f=r[u];if(o&&f[2]?f[1]!==n[f[0]]:!(f[0]in n))return false}for(;++u<i;){var f=r[u],c=f[0],a=n[c],l=f[1];if(o&&f[2]){if(a===T&&!(c in n))return false;
}else{if(f=new Zn,e)var s=e(a,l,c,n,t,f);if(s===T?!Mt(l,a,3,e,f):!s)return false}}return true}function Ft(n){return!(!du(n)||ci&&ci in n)&&(_u(n)?hi:dn).test(Te(n))}function Nt(n){return yu(n)&&"[object RegExp]"==Ot(n)}function Pt(n){return yu(n)&&"[object Set]"==vo(n)}function Zt(n){return yu(n)&&gu(n.length)&&!!Bn[Ot(n)]}function qt(n){return typeof n=="function"?n:null==n?$u:typeof n=="object"?ff(n)?Jt(n[0],n[1]):Ht(n):Zu(n)}function Vt(n){if(!ze(n))return Li(n);var t,r=[];for(t in Qu(n))oi.call(n,t)&&"constructor"!=t&&r.push(t);
return r}function Kt(n,t){return n<t}function Gt(n,t){var r=-1,e=su(n)?Ku(n.length):[];return uo(n,function(n,u,i){e[++r]=t(n,u,i)}),e}function Ht(n){var t=xe(n);return 1==t.length&&t[0][2]?We(t[0][0],t[0][1]):function(r){return r===n||$t(r,n,t)}}function Jt(n,t){return Ie(n)&&t===t&&!du(t)?We(Me(n),t):function(r){var e=Ru(r,n);return e===T&&e===t?zu(r,n):Mt(t,e,3)}}function Yt(n,t,r,e,u){n!==t&&oo(t,function(i,o){if(u||(u=new Zn),du(i)){var f=u,c=Le(n,o),a=Le(t,o),l=f.get(a);if(l)it(n,o,l);else{
var l=e?e(c,a,o+"",n,t,f):T,s=l===T;if(s){var h=ff(a),p=!h&&af(a),_=!h&&!p&&_f(a),l=a;h||p||_?ff(c)?l=c:hu(c)?l=Ur(c):p?(s=false,l=Ir(a,true)):_?(s=false,l=zr(a,true)):l=[]:xu(a)||of(a)?(l=c,of(c)?l=Ou(c):du(c)&&!_u(c)||(l=Ae(a))):s=false}s&&(f.set(a,l),Yt(l,a,r,e,f),f.delete(a)),it(n,o,l)}}else f=e?e(Le(n,o),i,o+"",n,t,u):T,f===T&&(f=i),it(n,o,f)},Bu)}function Qt(n,t){var r=n.length;if(r)return t+=0>t?r:0,Se(t,r)?n[t]:T}function Xt(n,t,r){var e=-1;return t=c(t.length?t:[$u],k(ye())),n=Gt(n,function(n){return{
a:c(t,function(t){return t(n)}),b:++e,c:n}}),w(n,function(n,t){var e;n:{e=-1;for(var u=n.a,i=t.a,o=u.length,f=r.length;++e<o;){var c=Wr(u[e],i[e]);if(c){e=e>=f?c:c*("desc"==r[e]?-1:1);break n}}e=n.b-t.b}return e})}function nr(n,t){return tr(n,t,function(t,r){return zu(n,r)})}function tr(n,t,r){for(var e=-1,u=t.length,i={};++e<u;){var o=t[e],f=kt(n,o);r(f,o)&&lr(i,Sr(o,n),f)}return i}function rr(n){return function(t){return kt(t,n)}}function er(n,t,r,e){var u=e?g:v,i=-1,o=t.length,f=n;for(n===t&&(t=Ur(t)),
r&&(f=c(n,k(r)));++i<o;)for(var a=0,l=t[i],l=r?r(l):l;-1<(a=u(f,l,a,e));)f!==n&&xi.call(f,a,1),xi.call(n,a,1);return n}function ur(n,t){for(var r=n?t.length:0,e=r-1;r--;){var u=t[r];if(r==e||u!==i){var i=u;Se(u)?xi.call(n,u,1):xr(n,u)}}}function ir(n,t){return n+Ii(Ti()*(t-n+1))}function or(n,t){var r="";if(!n||1>t||9007199254740991<t)return r;do t%2&&(r+=n),(t=Ii(t/2))&&(n+=n);while(t);return r}function fr(n,t){return xo(Be(n,t,$u),n+"")}function cr(n){return Qn(Uu(n))}function ar(n,t){var r=Uu(n);
return De(r,pt(t,0,r.length))}function lr(n,t,r,e){if(!du(n))return n;t=Sr(t,n);for(var u=-1,i=t.length,o=i-1,f=n;null!=f&&++u<i;){var c=Me(t[u]),a=r;if(u!=o){var l=f[c],a=e?e(l,c,f):T;a===T&&(a=du(l)?l:Se(t[u+1])?[]:{})}ot(f,c,a),f=f[c]}return n}function sr(n){return De(Uu(n))}function hr(n,t,r){var e=-1,u=n.length;for(0>t&&(t=-t>u?0:u+t),r=r>u?u:r,0>r&&(r+=u),u=t>r?0:r-t>>>0,t>>>=0,r=Ku(u);++e<u;)r[e]=n[e+t];return r}function pr(n,t){var r;return uo(n,function(n,e,u){return r=t(n,e,u),!r}),!!r}
function _r(n,t,r){var e=0,u=null==n?e:n.length;if(typeof t=="number"&&t===t&&2147483647>=u){for(;e<u;){var i=e+u>>>1,o=n[i];null!==o&&!wu(o)&&(r?o<=t:o<t)?e=i+1:u=i}return u}return vr(n,t,$u,r)}function vr(n,t,r,e){t=r(t);for(var u=0,i=null==n?0:n.length,o=t!==t,f=null===t,c=wu(t),a=t===T;u<i;){var l=Ii((u+i)/2),s=r(n[l]),h=s!==T,p=null===s,_=s===s,v=wu(s);(o?e||_:a?_&&(e||h):f?_&&h&&(e||!p):c?_&&h&&!p&&(e||!v):p||v?0:e?s<=t:s<t)?u=l+1:i=l}return Ci(i,4294967294)}function gr(n,t){for(var r=-1,e=n.length,u=0,i=[];++r<e;){
var o=n[r],f=t?t(o):o;if(!r||!lu(f,c)){var c=f;i[u++]=0===o?0:o}}return i}function dr(n){return typeof n=="number"?n:wu(n)?F:+n}function yr(n){if(typeof n=="string")return n;if(ff(n))return c(n,yr)+"";if(wu(n))return ro?ro.call(n):"";var t=n+"";return"0"==t&&1/n==-$?"-0":t}function br(n,t,r){var e=-1,u=o,i=n.length,c=true,a=[],l=a;if(r)c=false,u=f;else if(200<=i){if(u=t?null:so(n))return U(u);c=false,u=O,l=new Nn}else l=t?[]:a;n:for(;++e<i;){var s=n[e],h=t?t(s):s,s=r||0!==s?s:0;if(c&&h===h){for(var p=l.length;p--;)if(l[p]===h)continue n;
t&&l.push(h),a.push(s)}else u(l,h,r)||(l!==a&&l.push(h),a.push(s))}return a}function xr(n,t){return t=Sr(t,n),n=2>t.length?n:kt(n,hr(t,0,-1)),null==n||delete n[Me(Ve(t))]}function jr(n,t,r,e){for(var u=n.length,i=e?u:-1;(e?i--:++i<u)&&t(n[i],i,n););return r?hr(n,e?0:i,e?i+1:u):hr(n,e?i+1:0,e?u:i)}function wr(n,t){var r=n;return r instanceof Un&&(r=r.value()),l(t,function(n,t){return t.func.apply(t.thisArg,a([n],t.args))},r)}function mr(n,t,r){var e=n.length;if(2>e)return e?br(n[0]):[];for(var u=-1,i=Ku(e);++u<e;)for(var o=n[u],f=-1;++f<e;)f!=u&&(i[u]=yt(i[u]||o,n[f],t,r));
return br(wt(i,1),t,r)}function Ar(n,t,r){for(var e=-1,u=n.length,i=t.length,o={};++e<u;)r(o,n[e],e<i?t[e]:T);return o}function Er(n){return hu(n)?n:[]}function kr(n){return typeof n=="function"?n:$u}function Sr(n,t){return ff(n)?n:Ie(n,t)?[n]:jo(Iu(n))}function Or(n,t,r){var e=n.length;return r=r===T?e:r,!t&&r>=e?n:hr(n,t,r)}function Ir(n,t){if(t)return n.slice();var r=n.length,r=gi?gi(r):new n.constructor(r);return n.copy(r),r}function Rr(n){var t=new n.constructor(n.byteLength);return new vi(t).set(new vi(n)),
t}function zr(n,t){return new n.constructor(t?Rr(n.buffer):n.buffer,n.byteOffset,n.length)}function Wr(n,t){if(n!==t){var r=n!==T,e=null===n,u=n===n,i=wu(n),o=t!==T,f=null===t,c=t===t,a=wu(t);if(!f&&!a&&!i&&n>t||i&&o&&c&&!f&&!a||e&&o&&c||!r&&c||!u)return 1;if(!e&&!i&&!a&&n<t||a&&r&&u&&!e&&!i||f&&r&&u||!o&&u||!c)return-1}return 0}function Br(n,t,r,e){var u=-1,i=n.length,o=r.length,f=-1,c=t.length,a=Ui(i-o,0),l=Ku(c+a);for(e=!e;++f<c;)l[f]=t[f];for(;++u<o;)(e||u<i)&&(l[r[u]]=n[u]);for(;a--;)l[f++]=n[u++];
return l}function Lr(n,t,r,e){var u=-1,i=n.length,o=-1,f=r.length,c=-1,a=t.length,l=Ui(i-f,0),s=Ku(l+a);for(e=!e;++u<l;)s[u]=n[u];for(l=u;++c<a;)s[l+c]=t[c];for(;++o<f;)(e||u<i)&&(s[l+r[o]]=n[u++]);return s}function Ur(n,t){var r=-1,e=n.length;for(t||(t=Ku(e));++r<e;)t[r]=n[r];return t}function Cr(n,t,r,e){var u=!r;r||(r={});for(var i=-1,o=t.length;++i<o;){var f=t[i],c=e?e(r[f],n[f],f,r,n):T;c===T&&(c=n[f]),u?st(r,f,c):ot(r,f,c)}return r}function Dr(n,t){return Cr(n,po(n),t)}function Mr(n,t){return Cr(n,_o(n),t);
}function Tr(n,r){return function(e,u){var i=ff(e)?t:ct,o=r?r():{};return i(e,n,ye(u,2),o)}}function $r(n){return fr(function(t,r){var e=-1,u=r.length,i=1<u?r[u-1]:T,o=2<u?r[2]:T,i=3<n.length&&typeof i=="function"?(u--,i):T;for(o&&Oe(r[0],r[1],o)&&(i=3>u?T:i,u=1),t=Qu(t);++e<u;)(o=r[e])&&n(t,o,e,i);return t})}function Fr(n,t){return function(r,e){if(null==r)return r;if(!su(r))return n(r,e);for(var u=r.length,i=t?u:-1,o=Qu(r);(t?i--:++i<u)&&false!==e(o[i],i,o););return r}}function Nr(n){return function(t,r,e){
var u=-1,i=Qu(t);e=e(t);for(var o=e.length;o--;){var f=e[n?o:++u];if(false===r(i[f],f,i))break}return t}}function Pr(n,t,r){function e(){return(this&&this!==$n&&this instanceof e?i:n).apply(u?r:this,arguments)}var u=1&t,i=Vr(n);return e}function Zr(n){return function(t){t=Iu(t);var r=Rn.test(t)?M(t):T,e=r?r[0]:t.charAt(0);return t=r?Or(r,1).join(""):t.slice(1),e[n]()+t}}function qr(n){return function(t){return l(Mu(Du(t).replace(kn,"")),n,"")}}function Vr(n){return function(){var t=arguments;switch(t.length){
case 0:return new n;case 1:return new n(t[0]);case 2:return new n(t[0],t[1]);case 3:return new n(t[0],t[1],t[2]);case 4:return new n(t[0],t[1],t[2],t[3]);case 5:return new n(t[0],t[1],t[2],t[3],t[4]);case 6:return new n(t[0],t[1],t[2],t[3],t[4],t[5]);case 7:return new n(t[0],t[1],t[2],t[3],t[4],t[5],t[6])}var r=eo(n.prototype),t=n.apply(r,t);return du(t)?t:r}}function Kr(t,r,e){function u(){for(var o=arguments.length,f=Ku(o),c=o,a=de(u);c--;)f[c]=arguments[c];return c=3>o&&f[0]!==a&&f[o-1]!==a?[]:L(f,a),
o-=c.length,o<e?ue(t,r,Jr,u.placeholder,T,f,c,T,T,e-o):n(this&&this!==$n&&this instanceof u?i:t,this,f)}var i=Vr(t);return u}function Gr(n){return function(t,r,e){var u=Qu(t);if(!su(t)){var i=ye(r,3);t=Wu(t),r=function(n){return i(u[n],n,u)}}return r=n(t,r,e),-1<r?u[i?t[r]:r]:T}}function Hr(n){return pe(function(t){var r=t.length,e=r,u=On.prototype.thru;for(n&&t.reverse();e--;){var i=t[e];if(typeof i!="function")throw new ti("Expected a function");if(u&&!o&&"wrapper"==ge(i))var o=new On([],true)}for(e=o?e:r;++e<r;)var i=t[e],u=ge(i),f="wrapper"==u?ho(i):T,o=f&&Re(f[0])&&424==f[1]&&!f[4].length&&1==f[9]?o[ge(f[0])].apply(o,f[3]):1==i.length&&Re(i)?o[u]():o.thru(i);
return function(){var n=arguments,e=n[0];if(o&&1==n.length&&ff(e))return o.plant(e).value();for(var u=0,n=r?t[u].apply(this,n):e;++u<r;)n=t[u].call(this,n);return n}})}function Jr(n,t,r,e,u,i,o,f,c,a){function l(){for(var d=arguments.length,y=Ku(d),b=d;b--;)y[b]=arguments[b];if(_){var x,j=de(l),b=y.length;for(x=0;b--;)y[b]===j&&++x}if(e&&(y=Br(y,e,u,_)),i&&(y=Lr(y,i,o,_)),d-=x,_&&d<a)return j=L(y,j),ue(n,t,Jr,l.placeholder,r,y,j,f,c,a-d);if(j=h?r:this,b=p?j[n]:n,d=y.length,f){x=y.length;for(var w=Ci(f.length,x),m=Ur(y);w--;){
var A=f[w];y[w]=Se(A,x)?m[A]:T}}else v&&1<d&&y.reverse();return s&&c<d&&(y.length=c),this&&this!==$n&&this instanceof l&&(b=g||Vr(b)),b.apply(j,y)}var s=128&t,h=1&t,p=2&t,_=24&t,v=512&t,g=p?T:Vr(n);return l}function Yr(n,t){return function(r,e){return Bt(r,n,t(e))}}function Qr(n,t){return function(r,e){var u;if(r===T&&e===T)return t;if(r!==T&&(u=r),e!==T){if(u===T)return e;typeof r=="string"||typeof e=="string"?(r=yr(r),e=yr(e)):(r=dr(r),e=dr(e)),u=n(r,e)}return u}}function Xr(t){return pe(function(r){
return r=c(r,k(ye())),fr(function(e){var u=this;return t(r,function(t){return n(t,u,e)})})})}function ne(n,t){t=t===T?" ":yr(t);var r=t.length;return 2>r?r?or(t,n):t:(r=or(t,Oi(n/D(t))),Rn.test(t)?Or(M(r),0,n).join(""):r.slice(0,n))}function te(t,r,e,u){function i(){for(var r=-1,c=arguments.length,a=-1,l=u.length,s=Ku(l+c),h=this&&this!==$n&&this instanceof i?f:t;++a<l;)s[a]=u[a];for(;c--;)s[a++]=arguments[++r];return n(h,o?e:this,s)}var o=1&r,f=Vr(t);return i}function re(n){return function(t,r,e){
e&&typeof e!="number"&&Oe(t,r,e)&&(r=e=T),t=Au(t),r===T?(r=t,t=0):r=Au(r),e=e===T?t<r?1:-1:Au(e);var u=-1;r=Ui(Oi((r-t)/(e||1)),0);for(var i=Ku(r);r--;)i[n?r:++u]=t,t+=e;return i}}function ee(n){return function(t,r){return typeof t=="string"&&typeof r=="string"||(t=Su(t),r=Su(r)),n(t,r)}}function ue(n,t,r,e,u,i,o,f,c,a){var l=8&t,s=l?o:T;o=l?T:o;var h=l?i:T;return i=l?T:i,t=(t|(l?32:64))&~(l?64:32),4&t||(t&=-4),u=[n,t,u,h,s,i,o,f,c,a],r=r.apply(T,u),Re(n)&&yo(r,u),r.placeholder=e,Ue(r,n,t)}function ie(n){
var t=Yu[n];return function(n,r){if(n=Su(n),(r=null==r?0:Ci(Eu(r),292))&&Wi(n)){var e=(Iu(n)+"e").split("e"),e=t(e[0]+"e"+(+e[1]+r)),e=(Iu(e)+"e").split("e");return+(e[0]+"e"+(+e[1]-r))}return t(n)}}function oe(n){return function(t){var r=vo(t);return"[object Map]"==r?W(t):"[object Set]"==r?C(t):E(t,n(t))}}function fe(n,t,r,e,u,i,o,f){var c=2&t;if(!c&&typeof n!="function")throw new ti("Expected a function");var a=e?e.length:0;if(a||(t&=-97,e=u=T),o=o===T?o:Ui(Eu(o),0),f=f===T?f:Eu(f),a-=u?u.length:0,
64&t){var l=e,s=u;e=u=T}var h=c?T:ho(n);return i=[n,t,r,e,u,l,s,i,o,f],h&&(r=i[1],n=h[1],t=r|n,e=128==n&&8==r||128==n&&256==r&&i[7].length<=h[8]||384==n&&h[7].length<=h[8]&&8==r,131>t||e)&&(1&n&&(i[2]=h[2],t|=1&r?0:4),(r=h[3])&&(e=i[3],i[3]=e?Br(e,r,h[4]):r,i[4]=e?L(i[3],"__lodash_placeholder__"):h[4]),(r=h[5])&&(e=i[5],i[5]=e?Lr(e,r,h[6]):r,i[6]=e?L(i[5],"__lodash_placeholder__"):h[6]),(r=h[7])&&(i[7]=r),128&n&&(i[8]=null==i[8]?h[8]:Ci(i[8],h[8])),null==i[9]&&(i[9]=h[9]),i[0]=h[0],i[1]=t),n=i[0],
t=i[1],r=i[2],e=i[3],u=i[4],f=i[9]=i[9]===T?c?0:n.length:Ui(i[9]-a,0),!f&&24&t&&(t&=-25),Ue((h?co:yo)(t&&1!=t?8==t||16==t?Kr(n,t,f):32!=t&&33!=t||u.length?Jr.apply(T,i):te(n,t,r,e):Pr(n,t,r),i),n,t)}function ce(n,t,r,e){return n===T||lu(n,ei[r])&&!oi.call(e,r)?t:n}function ae(n,t,r,e,u,i){return du(n)&&du(t)&&(i.set(t,n),Yt(n,t,T,ae,i),i.delete(t)),n}function le(n){return xu(n)?T:n}function se(n,t,r,e,u,i){var o=1&r,f=n.length,c=t.length;if(f!=c&&!(o&&c>f))return false;if((c=i.get(n))&&i.get(t))return c==t;
var c=-1,a=true,l=2&r?new Nn:T;for(i.set(n,t),i.set(t,n);++c<f;){var s=n[c],p=t[c];if(e)var _=o?e(p,s,c,t,n,i):e(s,p,c,n,t,i);if(_!==T){if(_)continue;a=false;break}if(l){if(!h(t,function(n,t){if(!O(l,t)&&(s===n||u(s,n,r,e,i)))return l.push(t)})){a=false;break}}else if(s!==p&&!u(s,p,r,e,i)){a=false;break}}return i.delete(n),i.delete(t),a}function he(n,t,r,e,u,i,o){switch(r){case"[object DataView]":if(n.byteLength!=t.byteLength||n.byteOffset!=t.byteOffset)break;n=n.buffer,t=t.buffer;case"[object ArrayBuffer]":
if(n.byteLength!=t.byteLength||!i(new vi(n),new vi(t)))break;return true;case"[object Boolean]":case"[object Date]":case"[object Number]":return lu(+n,+t);case"[object Error]":return n.name==t.name&&n.message==t.message;case"[object RegExp]":case"[object String]":return n==t+"";case"[object Map]":var f=W;case"[object Set]":if(f||(f=U),n.size!=t.size&&!(1&e))break;return(r=o.get(n))?r==t:(e|=2,o.set(n,t),t=se(f(n),f(t),e,u,i,o),o.delete(n),t);case"[object Symbol]":if(to)return to.call(n)==to.call(t)}
return false}function pe(n){return xo(Be(n,T,Ze),n+"")}function _e(n){return St(n,Wu,po)}function ve(n){return St(n,Bu,_o)}function ge(n){for(var t=n.name+"",r=Gi[t],e=oi.call(Gi,t)?r.length:0;e--;){var u=r[e],i=u.func;if(null==i||i==n)return u.name}return t}function de(n){return(oi.call(An,"placeholder")?An:n).placeholder}function ye(){var n=An.iteratee||Fu,n=n===Fu?qt:n;return arguments.length?n(arguments[0],arguments[1]):n}function be(n,t){var r=n.__data__,e=typeof t;return("string"==e||"number"==e||"symbol"==e||"boolean"==e?"__proto__"!==t:null===t)?r[typeof t=="string"?"string":"hash"]:r.map;
}function xe(n){for(var t=Wu(n),r=t.length;r--;){var e=t[r],u=n[e];t[r]=[e,u,u===u&&!du(u)]}return t}function je(n,t){var r=null==n?T:n[t];return Ft(r)?r:T}function we(n,t,r){t=Sr(t,n);for(var e=-1,u=t.length,i=false;++e<u;){var o=Me(t[e]);if(!(i=null!=n&&r(n,o)))break;n=n[o]}return i||++e!=u?i:(u=null==n?0:n.length,!!u&&gu(u)&&Se(o,u)&&(ff(n)||of(n)))}function me(n){var t=n.length,r=new n.constructor(t);return t&&"string"==typeof n[0]&&oi.call(n,"index")&&(r.index=n.index,r.input=n.input),r}function Ae(n){
return typeof n.constructor!="function"||ze(n)?{}:eo(di(n))}function Ee(n,t,r){var e=n.constructor;switch(t){case"[object ArrayBuffer]":return Rr(n);case"[object Boolean]":case"[object Date]":return new e(+n);case"[object DataView]":return t=r?Rr(n.buffer):n.buffer,new n.constructor(t,n.byteOffset,n.byteLength);case"[object Float32Array]":case"[object Float64Array]":case"[object Int8Array]":case"[object Int16Array]":case"[object Int32Array]":case"[object Uint8Array]":case"[object Uint8ClampedArray]":
case"[object Uint16Array]":case"[object Uint32Array]":return zr(n,r);case"[object Map]":return new e;case"[object Number]":case"[object String]":return new e(n);case"[object RegExp]":return t=new n.constructor(n.source,_n.exec(n)),t.lastIndex=n.lastIndex,t;case"[object Set]":return new e;case"[object Symbol]":return to?Qu(to.call(n)):{}}}function ke(n){return ff(n)||of(n)||!!(ji&&n&&n[ji])}function Se(n,t){var r=typeof n;return t=null==t?9007199254740991:t,!!t&&("number"==r||"symbol"!=r&&bn.test(n))&&-1<n&&0==n%1&&n<t;
}function Oe(n,t,r){if(!du(r))return false;var e=typeof t;return!!("number"==e?su(r)&&Se(t,r.length):"string"==e&&t in r)&&lu(r[t],n)}function Ie(n,t){if(ff(n))return false;var r=typeof n;return!("number"!=r&&"symbol"!=r&&"boolean"!=r&&null!=n&&!wu(n))||(nn.test(n)||!X.test(n)||null!=t&&n in Qu(t))}function Re(n){var t=ge(n),r=An[t];return typeof r=="function"&&t in Un.prototype&&(n===r||(t=ho(r),!!t&&n===t[0]))}function ze(n){var t=n&&n.constructor;return n===(typeof t=="function"&&t.prototype||ei)}function We(n,t){
return function(r){return null!=r&&(r[n]===t&&(t!==T||n in Qu(r)))}}function Be(t,r,e){return r=Ui(r===T?t.length-1:r,0),function(){for(var u=arguments,i=-1,o=Ui(u.length-r,0),f=Ku(o);++i<o;)f[i]=u[r+i];for(i=-1,o=Ku(r+1);++i<r;)o[i]=u[i];return o[r]=e(f),n(t,this,o)}}function Le(n,t){if(("constructor"!==t||"function"!=typeof n[t])&&"__proto__"!=t)return n[t]}function Ue(n,t,r){var e=t+"";t=xo;var u,i=$e;return u=(u=e.match(an))?u[1].split(ln):[],r=i(u,r),(i=r.length)&&(u=i-1,r[u]=(1<i?"& ":"")+r[u],
r=r.join(2<i?", ":" "),e=e.replace(cn,"{\n/* [wrapped with "+r+"] */\n")),t(n,e)}function Ce(n){var t=0,r=0;return function(){var e=Di(),u=16-(e-r);if(r=e,0<u){if(800<=++t)return arguments[0]}else t=0;return n.apply(T,arguments)}}function De(n,t){var r=-1,e=n.length,u=e-1;for(t=t===T?e:t;++r<t;){var e=ir(r,u),i=n[e];n[e]=n[r],n[r]=i}return n.length=t,n}function Me(n){if(typeof n=="string"||wu(n))return n;var t=n+"";return"0"==t&&1/n==-$?"-0":t}function Te(n){if(null!=n){try{return ii.call(n)}catch(n){}
return n+""}return""}function $e(n,t){return r(N,function(r){var e="_."+r[0];t&r[1]&&!o(n,e)&&n.push(e)}),n.sort()}function Fe(n){if(n instanceof Un)return n.clone();var t=new On(n.__wrapped__,n.__chain__);return t.__actions__=Ur(n.__actions__),t.__index__=n.__index__,t.__values__=n.__values__,t}function Ne(n,t,r){var e=null==n?0:n.length;return e?(r=null==r?0:Eu(r),0>r&&(r=Ui(e+r,0)),_(n,ye(t,3),r)):-1}function Pe(n,t,r){var e=null==n?0:n.length;if(!e)return-1;var u=e-1;return r!==T&&(u=Eu(r),u=0>r?Ui(e+u,0):Ci(u,e-1)),
_(n,ye(t,3),u,true)}function Ze(n){return(null==n?0:n.length)?wt(n,1):[]}function qe(n){return n&&n.length?n[0]:T}function Ve(n){var t=null==n?0:n.length;return t?n[t-1]:T}function Ke(n,t){return n&&n.length&&t&&t.length?er(n,t):n}function Ge(n){return null==n?n:$i.call(n)}function He(n){if(!n||!n.length)return[];var t=0;return n=i(n,function(n){if(hu(n))return t=Ui(n.length,t),true}),A(t,function(t){return c(n,b(t))})}function Je(t,r){if(!t||!t.length)return[];var e=He(t);return null==r?e:c(e,function(t){
return n(r,T,t)})}function Ye(n){return n=An(n),n.__chain__=true,n}function Qe(n,t){return t(n)}function Xe(){return this}function nu(n,t){return(ff(n)?r:uo)(n,ye(t,3))}function tu(n,t){return(ff(n)?e:io)(n,ye(t,3))}function ru(n,t){return(ff(n)?c:Gt)(n,ye(t,3))}function eu(n,t,r){return t=r?T:t,t=n&&null==t?n.length:t,fe(n,128,T,T,T,T,t)}function uu(n,t){var r;if(typeof t!="function")throw new ti("Expected a function");return n=Eu(n),function(){return 0<--n&&(r=t.apply(this,arguments)),1>=n&&(t=T),
r}}function iu(n,t,r){return t=r?T:t,n=fe(n,8,T,T,T,T,T,t),n.placeholder=iu.placeholder,n}function ou(n,t,r){return t=r?T:t,n=fe(n,16,T,T,T,T,T,t),n.placeholder=ou.placeholder,n}function fu(n,t,r){function e(t){var r=c,e=a;return c=a=T,_=t,s=n.apply(e,r)}function u(n){var r=n-p;return n-=_,p===T||r>=t||0>r||g&&n>=l}function i(){var n=Go();if(u(n))return o(n);var r,e=bo;r=n-_,n=t-(n-p),r=g?Ci(n,l-r):n,h=e(i,r)}function o(n){return h=T,d&&c?e(n):(c=a=T,s)}function f(){var n=Go(),r=u(n);if(c=arguments,
a=this,p=n,r){if(h===T)return _=n=p,h=bo(i,t),v?e(n):s;if(g)return lo(h),h=bo(i,t),e(p)}return h===T&&(h=bo(i,t)),s}var c,a,l,s,h,p,_=0,v=false,g=false,d=true;if(typeof n!="function")throw new ti("Expected a function");return t=Su(t)||0,du(r)&&(v=!!r.leading,l=(g="maxWait"in r)?Ui(Su(r.maxWait)||0,t):l,d="trailing"in r?!!r.trailing:d),f.cancel=function(){h!==T&&lo(h),_=0,c=p=a=h=T},f.flush=function(){return h===T?s:o(Go())},f}function cu(n,t){function r(){var e=arguments,u=t?t.apply(this,e):e[0],i=r.cache;
return i.has(u)?i.get(u):(e=n.apply(this,e),r.cache=i.set(u,e)||i,e)}if(typeof n!="function"||null!=t&&typeof t!="function")throw new ti("Expected a function");return r.cache=new(cu.Cache||Fn),r}function au(n){if(typeof n!="function")throw new ti("Expected a function");return function(){var t=arguments;switch(t.length){case 0:return!n.call(this);case 1:return!n.call(this,t[0]);case 2:return!n.call(this,t[0],t[1]);case 3:return!n.call(this,t[0],t[1],t[2])}return!n.apply(this,t)}}function lu(n,t){return n===t||n!==n&&t!==t;
}function su(n){return null!=n&&gu(n.length)&&!_u(n)}function hu(n){return yu(n)&&su(n)}function pu(n){if(!yu(n))return false;var t=Ot(n);return"[object Error]"==t||"[object DOMException]"==t||typeof n.message=="string"&&typeof n.name=="string"&&!xu(n)}function _u(n){return!!du(n)&&(n=Ot(n),"[object Function]"==n||"[object GeneratorFunction]"==n||"[object AsyncFunction]"==n||"[object Proxy]"==n)}function vu(n){return typeof n=="number"&&n==Eu(n)}function gu(n){return typeof n=="number"&&-1<n&&0==n%1&&9007199254740991>=n;
}function du(n){var t=typeof n;return null!=n&&("object"==t||"function"==t)}function yu(n){return null!=n&&typeof n=="object"}function bu(n){return typeof n=="number"||yu(n)&&"[object Number]"==Ot(n)}function xu(n){return!(!yu(n)||"[object Object]"!=Ot(n))&&(n=di(n),null===n||(n=oi.call(n,"constructor")&&n.constructor,typeof n=="function"&&n instanceof n&&ii.call(n)==li))}function ju(n){return typeof n=="string"||!ff(n)&&yu(n)&&"[object String]"==Ot(n)}function wu(n){return typeof n=="symbol"||yu(n)&&"[object Symbol]"==Ot(n);
}function mu(n){if(!n)return[];if(su(n))return ju(n)?M(n):Ur(n);if(wi&&n[wi]){n=n[wi]();for(var t,r=[];!(t=n.next()).done;)r.push(t.value);return r}return t=vo(n),("[object Map]"==t?W:"[object Set]"==t?U:Uu)(n)}function Au(n){return n?(n=Su(n),n===$||n===-$?1.7976931348623157e308*(0>n?-1:1):n===n?n:0):0===n?n:0}function Eu(n){n=Au(n);var t=n%1;return n===n?t?n-t:n:0}function ku(n){return n?pt(Eu(n),0,4294967295):0}function Su(n){if(typeof n=="number")return n;if(wu(n))return F;if(du(n)&&(n=typeof n.valueOf=="function"?n.valueOf():n,
n=du(n)?n+"":n),typeof n!="string")return 0===n?n:+n;n=n.replace(un,"");var t=gn.test(n);return t||yn.test(n)?Dn(n.slice(2),t?2:8):vn.test(n)?F:+n}function Ou(n){return Cr(n,Bu(n))}function Iu(n){return null==n?"":yr(n)}function Ru(n,t,r){return n=null==n?T:kt(n,t),n===T?r:n}function zu(n,t){return null!=n&&we(n,t,zt)}function Wu(n){return su(n)?qn(n):Vt(n)}function Bu(n){if(su(n))n=qn(n,true);else if(du(n)){var t,r=ze(n),e=[];for(t in n)("constructor"!=t||!r&&oi.call(n,t))&&e.push(t);n=e}else{if(t=[],
null!=n)for(r in Qu(n))t.push(r);n=t}return n}function Lu(n,t){if(null==n)return{};var r=c(ve(n),function(n){return[n]});return t=ye(t),tr(n,r,function(n,r){return t(n,r[0])})}function Uu(n){return null==n?[]:S(n,Wu(n))}function Cu(n){return $f(Iu(n).toLowerCase())}function Du(n){return(n=Iu(n))&&n.replace(xn,Xn).replace(Sn,"")}function Mu(n,t,r){return n=Iu(n),t=r?T:t,t===T?zn.test(n)?n.match(In)||[]:n.match(sn)||[]:n.match(t)||[]}function Tu(n){return function(){return n}}function $u(n){return n;
}function Fu(n){return qt(typeof n=="function"?n:_t(n,1))}function Nu(n,t,e){var u=Wu(t),i=Et(t,u);null!=e||du(t)&&(i.length||!u.length)||(e=t,t=n,n=this,i=Et(t,Wu(t)));var o=!(du(e)&&"chain"in e&&!e.chain),f=_u(n);return r(i,function(r){var e=t[r];n[r]=e,f&&(n.prototype[r]=function(){var t=this.__chain__;if(o||t){var r=n(this.__wrapped__);return(r.__actions__=Ur(this.__actions__)).push({func:e,args:arguments,thisArg:n}),r.__chain__=t,r}return e.apply(n,a([this.value()],arguments))})}),n}function Pu(){}
function Zu(n){return Ie(n)?b(Me(n)):rr(n)}function qu(){return[]}function Vu(){return false}mn=null==mn?$n:rt.defaults($n.Object(),mn,rt.pick($n,Wn));var Ku=mn.Array,Gu=mn.Date,Hu=mn.Error,Ju=mn.Function,Yu=mn.Math,Qu=mn.Object,Xu=mn.RegExp,ni=mn.String,ti=mn.TypeError,ri=Ku.prototype,ei=Qu.prototype,ui=mn["__core-js_shared__"],ii=Ju.prototype.toString,oi=ei.hasOwnProperty,fi=0,ci=function(){var n=/[^.]+$/.exec(ui&&ui.keys&&ui.keys.IE_PROTO||"");return n?"Symbol(src)_1."+n:""}(),ai=ei.toString,li=ii.call(Qu),si=$n._,hi=Xu("^"+ii.call(oi).replace(rn,"\\$&").replace(/hasOwnProperty|(function).*?(?=\\\()| for .+?(?=\\\])/g,"$1.*?")+"$"),pi=Pn?mn.Buffer:T,_i=mn.Symbol,vi=mn.Uint8Array,gi=pi?pi.g:T,di=B(Qu.getPrototypeOf,Qu),yi=Qu.create,bi=ei.propertyIsEnumerable,xi=ri.splice,ji=_i?_i.isConcatSpreadable:T,wi=_i?_i.iterator:T,mi=_i?_i.toStringTag:T,Ai=function(){
try{var n=je(Qu,"defineProperty");return n({},"",{}),n}catch(n){}}(),Ei=mn.clearTimeout!==$n.clearTimeout&&mn.clearTimeout,ki=Gu&&Gu.now!==$n.Date.now&&Gu.now,Si=mn.setTimeout!==$n.setTimeout&&mn.setTimeout,Oi=Yu.ceil,Ii=Yu.floor,Ri=Qu.getOwnPropertySymbols,zi=pi?pi.isBuffer:T,Wi=mn.isFinite,Bi=ri.join,Li=B(Qu.keys,Qu),Ui=Yu.max,Ci=Yu.min,Di=Gu.now,Mi=mn.parseInt,Ti=Yu.random,$i=ri.reverse,Fi=je(mn,"DataView"),Ni=je(mn,"Map"),Pi=je(mn,"Promise"),Zi=je(mn,"Set"),qi=je(mn,"WeakMap"),Vi=je(Qu,"create"),Ki=qi&&new qi,Gi={},Hi=Te(Fi),Ji=Te(Ni),Yi=Te(Pi),Qi=Te(Zi),Xi=Te(qi),no=_i?_i.prototype:T,to=no?no.valueOf:T,ro=no?no.toString:T,eo=function(){
function n(){}return function(t){return du(t)?yi?yi(t):(n.prototype=t,t=new n,n.prototype=T,t):{}}}();An.templateSettings={escape:J,evaluate:Y,interpolate:Q,variable:"",imports:{_:An}},An.prototype=En.prototype,An.prototype.constructor=An,On.prototype=eo(En.prototype),On.prototype.constructor=On,Un.prototype=eo(En.prototype),Un.prototype.constructor=Un,Mn.prototype.clear=function(){this.__data__=Vi?Vi(null):{},this.size=0},Mn.prototype.delete=function(n){return n=this.has(n)&&delete this.__data__[n],
this.size-=n?1:0,n},Mn.prototype.get=function(n){var t=this.__data__;return Vi?(n=t[n],"__lodash_hash_undefined__"===n?T:n):oi.call(t,n)?t[n]:T},Mn.prototype.has=function(n){var t=this.__data__;return Vi?t[n]!==T:oi.call(t,n)},Mn.prototype.set=function(n,t){var r=this.__data__;return this.size+=this.has(n)?0:1,r[n]=Vi&&t===T?"__lodash_hash_undefined__":t,this},Tn.prototype.clear=function(){this.__data__=[],this.size=0},Tn.prototype.delete=function(n){var t=this.__data__;return n=ft(t,n),!(0>n)&&(n==t.length-1?t.pop():xi.call(t,n,1),
--this.size,true)},Tn.prototype.get=function(n){var t=this.__data__;return n=ft(t,n),0>n?T:t[n][1]},Tn.prototype.has=function(n){return-1<ft(this.__data__,n)},Tn.prototype.set=function(n,t){var r=this.__data__,e=ft(r,n);return 0>e?(++this.size,r.push([n,t])):r[e][1]=t,this},Fn.prototype.clear=function(){this.size=0,this.__data__={hash:new Mn,map:new(Ni||Tn),string:new Mn}},Fn.prototype.delete=function(n){return n=be(this,n).delete(n),this.size-=n?1:0,n},Fn.prototype.get=function(n){return be(this,n).get(n);
},Fn.prototype.has=function(n){return be(this,n).has(n)},Fn.prototype.set=function(n,t){var r=be(this,n),e=r.size;return r.set(n,t),this.size+=r.size==e?0:1,this},Nn.prototype.add=Nn.prototype.push=function(n){return this.__data__.set(n,"__lodash_hash_undefined__"),this},Nn.prototype.has=function(n){return this.__data__.has(n)},Zn.prototype.clear=function(){this.__data__=new Tn,this.size=0},Zn.prototype.delete=function(n){var t=this.__data__;return n=t.delete(n),this.size=t.size,n},Zn.prototype.get=function(n){
return this.__data__.get(n)},Zn.prototype.has=function(n){return this.__data__.has(n)},Zn.prototype.set=function(n,t){var r=this.__data__;if(r instanceof Tn){var e=r.__data__;if(!Ni||199>e.length)return e.push([n,t]),this.size=++r.size,this;r=this.__data__=new Fn(e)}return r.set(n,t),this.size=r.size,this};var uo=Fr(mt),io=Fr(At,true),oo=Nr(),fo=Nr(true),co=Ki?function(n,t){return Ki.set(n,t),n}:$u,ao=Ai?function(n,t){return Ai(n,"toString",{configurable:true,enumerable:false,value:Tu(t),writable:true})}:$u,lo=Ei||function(n){
return $n.clearTimeout(n)},so=Zi&&1/U(new Zi([,-0]))[1]==$?function(n){return new Zi(n)}:Pu,ho=Ki?function(n){return Ki.get(n)}:Pu,po=Ri?function(n){return null==n?[]:(n=Qu(n),i(Ri(n),function(t){return bi.call(n,t)}))}:qu,_o=Ri?function(n){for(var t=[];n;)a(t,po(n)),n=di(n);return t}:qu,vo=Ot;(Fi&&"[object DataView]"!=vo(new Fi(new ArrayBuffer(1)))||Ni&&"[object Map]"!=vo(new Ni)||Pi&&"[object Promise]"!=vo(Pi.resolve())||Zi&&"[object Set]"!=vo(new Zi)||qi&&"[object WeakMap]"!=vo(new qi))&&(vo=function(n){
var t=Ot(n);if(n=(n="[object Object]"==t?n.constructor:T)?Te(n):"")switch(n){case Hi:return"[object DataView]";case Ji:return"[object Map]";case Yi:return"[object Promise]";case Qi:return"[object Set]";case Xi:return"[object WeakMap]"}return t});var go=ui?_u:Vu,yo=Ce(co),bo=Si||function(n,t){return $n.setTimeout(n,t)},xo=Ce(ao),jo=function(n){n=cu(n,function(n){return 500===t.size&&t.clear(),n});var t=n.cache;return n}(function(n){var t=[];return 46===n.charCodeAt(0)&&t.push(""),n.replace(tn,function(n,r,e,u){
t.push(e?u.replace(hn,"$1"):r||n)}),t}),wo=fr(function(n,t){return hu(n)?yt(n,wt(t,1,hu,true)):[]}),mo=fr(function(n,t){var r=Ve(t);return hu(r)&&(r=T),hu(n)?yt(n,wt(t,1,hu,true),ye(r,2)):[]}),Ao=fr(function(n,t){var r=Ve(t);return hu(r)&&(r=T),hu(n)?yt(n,wt(t,1,hu,true),T,r):[]}),Eo=fr(function(n){var t=c(n,Er);return t.length&&t[0]===n[0]?Wt(t):[]}),ko=fr(function(n){var t=Ve(n),r=c(n,Er);return t===Ve(r)?t=T:r.pop(),r.length&&r[0]===n[0]?Wt(r,ye(t,2)):[]}),So=fr(function(n){var t=Ve(n),r=c(n,Er);return(t=typeof t=="function"?t:T)&&r.pop(),
r.length&&r[0]===n[0]?Wt(r,T,t):[]}),Oo=fr(Ke),Io=pe(function(n,t){var r=null==n?0:n.length,e=ht(n,t);return ur(n,c(t,function(n){return Se(n,r)?+n:n}).sort(Wr)),e}),Ro=fr(function(n){return br(wt(n,1,hu,true))}),zo=fr(function(n){var t=Ve(n);return hu(t)&&(t=T),br(wt(n,1,hu,true),ye(t,2))}),Wo=fr(function(n){var t=Ve(n),t=typeof t=="function"?t:T;return br(wt(n,1,hu,true),T,t)}),Bo=fr(function(n,t){return hu(n)?yt(n,t):[]}),Lo=fr(function(n){return mr(i(n,hu))}),Uo=fr(function(n){var t=Ve(n);return hu(t)&&(t=T),
mr(i(n,hu),ye(t,2))}),Co=fr(function(n){var t=Ve(n),t=typeof t=="function"?t:T;return mr(i(n,hu),T,t)}),Do=fr(He),Mo=fr(function(n){var t=n.length,t=1<t?n[t-1]:T,t=typeof t=="function"?(n.pop(),t):T;return Je(n,t)}),To=pe(function(n){function t(t){return ht(t,n)}var r=n.length,e=r?n[0]:0,u=this.__wrapped__;return!(1<r||this.__actions__.length)&&u instanceof Un&&Se(e)?(u=u.slice(e,+e+(r?1:0)),u.__actions__.push({func:Qe,args:[t],thisArg:T}),new On(u,this.__chain__).thru(function(n){return r&&!n.length&&n.push(T),
n})):this.thru(t)}),$o=Tr(function(n,t,r){oi.call(n,r)?++n[r]:st(n,r,1)}),Fo=Gr(Ne),No=Gr(Pe),Po=Tr(function(n,t,r){oi.call(n,r)?n[r].push(t):st(n,r,[t])}),Zo=fr(function(t,r,e){var u=-1,i=typeof r=="function",o=su(t)?Ku(t.length):[];return uo(t,function(t){o[++u]=i?n(r,t,e):Lt(t,r,e)}),o}),qo=Tr(function(n,t,r){st(n,r,t)}),Vo=Tr(function(n,t,r){n[r?0:1].push(t)},function(){return[[],[]]}),Ko=fr(function(n,t){if(null==n)return[];var r=t.length;return 1<r&&Oe(n,t[0],t[1])?t=[]:2<r&&Oe(t[0],t[1],t[2])&&(t=[t[0]]),
Xt(n,wt(t,1),[])}),Go=ki||function(){return $n.Date.now()},Ho=fr(function(n,t,r){var e=1;if(r.length)var u=L(r,de(Ho)),e=32|e;return fe(n,e,t,r,u)}),Jo=fr(function(n,t,r){var e=3;if(r.length)var u=L(r,de(Jo)),e=32|e;return fe(t,e,n,r,u)}),Yo=fr(function(n,t){return dt(n,1,t)}),Qo=fr(function(n,t,r){return dt(n,Su(t)||0,r)});cu.Cache=Fn;var Xo=fr(function(t,r){r=1==r.length&&ff(r[0])?c(r[0],k(ye())):c(wt(r,1),k(ye()));var e=r.length;return fr(function(u){for(var i=-1,o=Ci(u.length,e);++i<o;)u[i]=r[i].call(this,u[i]);
return n(t,this,u)})}),nf=fr(function(n,t){return fe(n,32,T,t,L(t,de(nf)))}),tf=fr(function(n,t){return fe(n,64,T,t,L(t,de(tf)))}),rf=pe(function(n,t){return fe(n,256,T,T,T,t)}),ef=ee(It),uf=ee(function(n,t){return n>=t}),of=Ut(function(){return arguments}())?Ut:function(n){return yu(n)&&oi.call(n,"callee")&&!bi.call(n,"callee")},ff=Ku.isArray,cf=Vn?k(Vn):Ct,af=zi||Vu,lf=Kn?k(Kn):Dt,sf=Gn?k(Gn):Tt,hf=Hn?k(Hn):Nt,pf=Jn?k(Jn):Pt,_f=Yn?k(Yn):Zt,vf=ee(Kt),gf=ee(function(n,t){return n<=t}),df=$r(function(n,t){
if(ze(t)||su(t))Cr(t,Wu(t),n);else for(var r in t)oi.call(t,r)&&ot(n,r,t[r])}),yf=$r(function(n,t){Cr(t,Bu(t),n)}),bf=$r(function(n,t,r,e){Cr(t,Bu(t),n,e)}),xf=$r(function(n,t,r,e){Cr(t,Wu(t),n,e)}),jf=pe(ht),wf=fr(function(n,t){n=Qu(n);var r=-1,e=t.length,u=2<e?t[2]:T;for(u&&Oe(t[0],t[1],u)&&(e=1);++r<e;)for(var u=t[r],i=Bu(u),o=-1,f=i.length;++o<f;){var c=i[o],a=n[c];(a===T||lu(a,ei[c])&&!oi.call(n,c))&&(n[c]=u[c])}return n}),mf=fr(function(t){return t.push(T,ae),n(Of,T,t)}),Af=Yr(function(n,t,r){
null!=t&&typeof t.toString!="function"&&(t=ai.call(t)),n[t]=r},Tu($u)),Ef=Yr(function(n,t,r){null!=t&&typeof t.toString!="function"&&(t=ai.call(t)),oi.call(n,t)?n[t].push(r):n[t]=[r]},ye),kf=fr(Lt),Sf=$r(function(n,t,r){Yt(n,t,r)}),Of=$r(function(n,t,r,e){Yt(n,t,r,e)}),If=pe(function(n,t){var r={};if(null==n)return r;var e=false;t=c(t,function(t){return t=Sr(t,n),e||(e=1<t.length),t}),Cr(n,ve(n),r),e&&(r=_t(r,7,le));for(var u=t.length;u--;)xr(r,t[u]);return r}),Rf=pe(function(n,t){return null==n?{}:nr(n,t);
}),zf=oe(Wu),Wf=oe(Bu),Bf=qr(function(n,t,r){return t=t.toLowerCase(),n+(r?Cu(t):t)}),Lf=qr(function(n,t,r){return n+(r?"-":"")+t.toLowerCase()}),Uf=qr(function(n,t,r){return n+(r?" ":"")+t.toLowerCase()}),Cf=Zr("toLowerCase"),Df=qr(function(n,t,r){return n+(r?"_":"")+t.toLowerCase()}),Mf=qr(function(n,t,r){return n+(r?" ":"")+$f(t)}),Tf=qr(function(n,t,r){return n+(r?" ":"")+t.toUpperCase()}),$f=Zr("toUpperCase"),Ff=fr(function(t,r){try{return n(t,T,r)}catch(n){return pu(n)?n:new Hu(n)}}),Nf=pe(function(n,t){
return r(t,function(t){t=Me(t),st(n,t,Ho(n[t],n))}),n}),Pf=Hr(),Zf=Hr(true),qf=fr(function(n,t){return function(r){return Lt(r,n,t)}}),Vf=fr(function(n,t){return function(r){return Lt(n,r,t)}}),Kf=Xr(c),Gf=Xr(u),Hf=Xr(h),Jf=re(),Yf=re(true),Qf=Qr(function(n,t){return n+t},0),Xf=ie("ceil"),nc=Qr(function(n,t){return n/t},1),tc=ie("floor"),rc=Qr(function(n,t){return n*t},1),ec=ie("round"),uc=Qr(function(n,t){return n-t},0);return An.after=function(n,t){if(typeof t!="function")throw new ti("Expected a function");
return n=Eu(n),function(){if(1>--n)return t.apply(this,arguments)}},An.ary=eu,An.assign=df,An.assignIn=yf,An.assignInWith=bf,An.assignWith=xf,An.at=jf,An.before=uu,An.bind=Ho,An.bindAll=Nf,An.bindKey=Jo,An.castArray=function(){if(!arguments.length)return[];var n=arguments[0];return ff(n)?n:[n]},An.chain=Ye,An.chunk=function(n,t,r){if(t=(r?Oe(n,t,r):t===T)?1:Ui(Eu(t),0),r=null==n?0:n.length,!r||1>t)return[];for(var e=0,u=0,i=Ku(Oi(r/t));e<r;)i[u++]=hr(n,e,e+=t);return i},An.compact=function(n){for(var t=-1,r=null==n?0:n.length,e=0,u=[];++t<r;){
var i=n[t];i&&(u[e++]=i)}return u},An.concat=function(){var n=arguments.length;if(!n)return[];for(var t=Ku(n-1),r=arguments[0];n--;)t[n-1]=arguments[n];return a(ff(r)?Ur(r):[r],wt(t,1))},An.cond=function(t){var r=null==t?0:t.length,e=ye();return t=r?c(t,function(n){if("function"!=typeof n[1])throw new ti("Expected a function");return[e(n[0]),n[1]]}):[],fr(function(e){for(var u=-1;++u<r;){var i=t[u];if(n(i[0],this,e))return n(i[1],this,e)}})},An.conforms=function(n){return vt(_t(n,1))},An.constant=Tu,
An.countBy=$o,An.create=function(n,t){var r=eo(n);return null==t?r:at(r,t)},An.curry=iu,An.curryRight=ou,An.debounce=fu,An.defaults=wf,An.defaultsDeep=mf,An.defer=Yo,An.delay=Qo,An.difference=wo,An.differenceBy=mo,An.differenceWith=Ao,An.drop=function(n,t,r){var e=null==n?0:n.length;return e?(t=r||t===T?1:Eu(t),hr(n,0>t?0:t,e)):[]},An.dropRight=function(n,t,r){var e=null==n?0:n.length;return e?(t=r||t===T?1:Eu(t),t=e-t,hr(n,0,0>t?0:t)):[]},An.dropRightWhile=function(n,t){return n&&n.length?jr(n,ye(t,3),true,true):[];
},An.dropWhile=function(n,t){return n&&n.length?jr(n,ye(t,3),true):[]},An.fill=function(n,t,r,e){var u=null==n?0:n.length;if(!u)return[];for(r&&typeof r!="number"&&Oe(n,t,r)&&(r=0,e=u),u=n.length,r=Eu(r),0>r&&(r=-r>u?0:u+r),e=e===T||e>u?u:Eu(e),0>e&&(e+=u),e=r>e?0:ku(e);r<e;)n[r++]=t;return n},An.filter=function(n,t){return(ff(n)?i:jt)(n,ye(t,3))},An.flatMap=function(n,t){return wt(ru(n,t),1)},An.flatMapDeep=function(n,t){return wt(ru(n,t),$)},An.flatMapDepth=function(n,t,r){return r=r===T?1:Eu(r),
wt(ru(n,t),r)},An.flatten=Ze,An.flattenDeep=function(n){return(null==n?0:n.length)?wt(n,$):[]},An.flattenDepth=function(n,t){return null!=n&&n.length?(t=t===T?1:Eu(t),wt(n,t)):[]},An.flip=function(n){return fe(n,512)},An.flow=Pf,An.flowRight=Zf,An.fromPairs=function(n){for(var t=-1,r=null==n?0:n.length,e={};++t<r;){var u=n[t];e[u[0]]=u[1]}return e},An.functions=function(n){return null==n?[]:Et(n,Wu(n))},An.functionsIn=function(n){return null==n?[]:Et(n,Bu(n))},An.groupBy=Po,An.initial=function(n){
return(null==n?0:n.length)?hr(n,0,-1):[]},An.intersection=Eo,An.intersectionBy=ko,An.intersectionWith=So,An.invert=Af,An.invertBy=Ef,An.invokeMap=Zo,An.iteratee=Fu,An.keyBy=qo,An.keys=Wu,An.keysIn=Bu,An.map=ru,An.mapKeys=function(n,t){var r={};return t=ye(t,3),mt(n,function(n,e,u){st(r,t(n,e,u),n)}),r},An.mapValues=function(n,t){var r={};return t=ye(t,3),mt(n,function(n,e,u){st(r,e,t(n,e,u))}),r},An.matches=function(n){return Ht(_t(n,1))},An.matchesProperty=function(n,t){return Jt(n,_t(t,1))},An.memoize=cu,
An.merge=Sf,An.mergeWith=Of,An.method=qf,An.methodOf=Vf,An.mixin=Nu,An.negate=au,An.nthArg=function(n){return n=Eu(n),fr(function(t){return Qt(t,n)})},An.omit=If,An.omitBy=function(n,t){return Lu(n,au(ye(t)))},An.once=function(n){return uu(2,n)},An.orderBy=function(n,t,r,e){return null==n?[]:(ff(t)||(t=null==t?[]:[t]),r=e?T:r,ff(r)||(r=null==r?[]:[r]),Xt(n,t,r))},An.over=Kf,An.overArgs=Xo,An.overEvery=Gf,An.overSome=Hf,An.partial=nf,An.partialRight=tf,An.partition=Vo,An.pick=Rf,An.pickBy=Lu,An.property=Zu,
An.propertyOf=function(n){return function(t){return null==n?T:kt(n,t)}},An.pull=Oo,An.pullAll=Ke,An.pullAllBy=function(n,t,r){return n&&n.length&&t&&t.length?er(n,t,ye(r,2)):n},An.pullAllWith=function(n,t,r){return n&&n.length&&t&&t.length?er(n,t,T,r):n},An.pullAt=Io,An.range=Jf,An.rangeRight=Yf,An.rearg=rf,An.reject=function(n,t){return(ff(n)?i:jt)(n,au(ye(t,3)))},An.remove=function(n,t){var r=[];if(!n||!n.length)return r;var e=-1,u=[],i=n.length;for(t=ye(t,3);++e<i;){var o=n[e];t(o,e,n)&&(r.push(o),
u.push(e))}return ur(n,u),r},An.rest=function(n,t){if(typeof n!="function")throw new ti("Expected a function");return t=t===T?t:Eu(t),fr(n,t)},An.reverse=Ge,An.sampleSize=function(n,t,r){return t=(r?Oe(n,t,r):t===T)?1:Eu(t),(ff(n)?et:ar)(n,t)},An.set=function(n,t,r){return null==n?n:lr(n,t,r)},An.setWith=function(n,t,r,e){return e=typeof e=="function"?e:T,null==n?n:lr(n,t,r,e)},An.shuffle=function(n){return(ff(n)?ut:sr)(n)},An.slice=function(n,t,r){var e=null==n?0:n.length;return e?(r&&typeof r!="number"&&Oe(n,t,r)?(t=0,
r=e):(t=null==t?0:Eu(t),r=r===T?e:Eu(r)),hr(n,t,r)):[]},An.sortBy=Ko,An.sortedUniq=function(n){return n&&n.length?gr(n):[]},An.sortedUniqBy=function(n,t){return n&&n.length?gr(n,ye(t,2)):[]},An.split=function(n,t,r){return r&&typeof r!="number"&&Oe(n,t,r)&&(t=r=T),r=r===T?4294967295:r>>>0,r?(n=Iu(n))&&(typeof t=="string"||null!=t&&!hf(t))&&(t=yr(t),!t&&Rn.test(n))?Or(M(n),0,r):n.split(t,r):[]},An.spread=function(t,r){if(typeof t!="function")throw new ti("Expected a function");return r=null==r?0:Ui(Eu(r),0),
fr(function(e){var u=e[r];return e=Or(e,0,r),u&&a(e,u),n(t,this,e)})},An.tail=function(n){var t=null==n?0:n.length;return t?hr(n,1,t):[]},An.take=function(n,t,r){return n&&n.length?(t=r||t===T?1:Eu(t),hr(n,0,0>t?0:t)):[]},An.takeRight=function(n,t,r){var e=null==n?0:n.length;return e?(t=r||t===T?1:Eu(t),t=e-t,hr(n,0>t?0:t,e)):[]},An.takeRightWhile=function(n,t){return n&&n.length?jr(n,ye(t,3),false,true):[]},An.takeWhile=function(n,t){return n&&n.length?jr(n,ye(t,3)):[]},An.tap=function(n,t){return t(n),
n},An.throttle=function(n,t,r){var e=true,u=true;if(typeof n!="function")throw new ti("Expected a function");return du(r)&&(e="leading"in r?!!r.leading:e,u="trailing"in r?!!r.trailing:u),fu(n,t,{leading:e,maxWait:t,trailing:u})},An.thru=Qe,An.toArray=mu,An.toPairs=zf,An.toPairsIn=Wf,An.toPath=function(n){return ff(n)?c(n,Me):wu(n)?[n]:Ur(jo(Iu(n)))},An.toPlainObject=Ou,An.transform=function(n,t,e){var u=ff(n),i=u||af(n)||_f(n);if(t=ye(t,4),null==e){var o=n&&n.constructor;e=i?u?new o:[]:du(n)&&_u(o)?eo(di(n)):{};
}return(i?r:mt)(n,function(n,r,u){return t(e,n,r,u)}),e},An.unary=function(n){return eu(n,1)},An.union=Ro,An.unionBy=zo,An.unionWith=Wo,An.uniq=function(n){return n&&n.length?br(n):[]},An.uniqBy=function(n,t){return n&&n.length?br(n,ye(t,2)):[]},An.uniqWith=function(n,t){return t=typeof t=="function"?t:T,n&&n.length?br(n,T,t):[]},An.unset=function(n,t){return null==n||xr(n,t)},An.unzip=He,An.unzipWith=Je,An.update=function(n,t,r){return null==n?n:lr(n,t,kr(r)(kt(n,t)),void 0)},An.updateWith=function(n,t,r,e){
return e=typeof e=="function"?e:T,null!=n&&(n=lr(n,t,kr(r)(kt(n,t)),e)),n},An.values=Uu,An.valuesIn=function(n){return null==n?[]:S(n,Bu(n))},An.without=Bo,An.words=Mu,An.wrap=function(n,t){return nf(kr(t),n)},An.xor=Lo,An.xorBy=Uo,An.xorWith=Co,An.zip=Do,An.zipObject=function(n,t){return Ar(n||[],t||[],ot)},An.zipObjectDeep=function(n,t){return Ar(n||[],t||[],lr)},An.zipWith=Mo,An.entries=zf,An.entriesIn=Wf,An.extend=yf,An.extendWith=bf,Nu(An,An),An.add=Qf,An.attempt=Ff,An.camelCase=Bf,An.capitalize=Cu,
An.ceil=Xf,An.clamp=function(n,t,r){return r===T&&(r=t,t=T),r!==T&&(r=Su(r),r=r===r?r:0),t!==T&&(t=Su(t),t=t===t?t:0),pt(Su(n),t,r)},An.clone=function(n){return _t(n,4)},An.cloneDeep=function(n){return _t(n,5)},An.cloneDeepWith=function(n,t){return t=typeof t=="function"?t:T,_t(n,5,t)},An.cloneWith=function(n,t){return t=typeof t=="function"?t:T,_t(n,4,t)},An.conformsTo=function(n,t){return null==t||gt(n,t,Wu(t))},An.deburr=Du,An.defaultTo=function(n,t){return null==n||n!==n?t:n},An.divide=nc,An.endsWith=function(n,t,r){
n=Iu(n),t=yr(t);var e=n.length,e=r=r===T?e:pt(Eu(r),0,e);return r-=t.length,0<=r&&n.slice(r,e)==t},An.eq=lu,An.escape=function(n){return(n=Iu(n))&&H.test(n)?n.replace(K,nt):n},An.escapeRegExp=function(n){return(n=Iu(n))&&en.test(n)?n.replace(rn,"\\$&"):n},An.every=function(n,t,r){var e=ff(n)?u:bt;return r&&Oe(n,t,r)&&(t=T),e(n,ye(t,3))},An.find=Fo,An.findIndex=Ne,An.findKey=function(n,t){return p(n,ye(t,3),mt)},An.findLast=No,An.findLastIndex=Pe,An.findLastKey=function(n,t){return p(n,ye(t,3),At);
},An.floor=tc,An.forEach=nu,An.forEachRight=tu,An.forIn=function(n,t){return null==n?n:oo(n,ye(t,3),Bu)},An.forInRight=function(n,t){return null==n?n:fo(n,ye(t,3),Bu)},An.forOwn=function(n,t){return n&&mt(n,ye(t,3))},An.forOwnRight=function(n,t){return n&&At(n,ye(t,3))},An.get=Ru,An.gt=ef,An.gte=uf,An.has=function(n,t){return null!=n&&we(n,t,Rt)},An.hasIn=zu,An.head=qe,An.identity=$u,An.includes=function(n,t,r,e){return n=su(n)?n:Uu(n),r=r&&!e?Eu(r):0,e=n.length,0>r&&(r=Ui(e+r,0)),ju(n)?r<=e&&-1<n.indexOf(t,r):!!e&&-1<v(n,t,r);
},An.indexOf=function(n,t,r){var e=null==n?0:n.length;return e?(r=null==r?0:Eu(r),0>r&&(r=Ui(e+r,0)),v(n,t,r)):-1},An.inRange=function(n,t,r){return t=Au(t),r===T?(r=t,t=0):r=Au(r),n=Su(n),n>=Ci(t,r)&&n<Ui(t,r)},An.invoke=kf,An.isArguments=of,An.isArray=ff,An.isArrayBuffer=cf,An.isArrayLike=su,An.isArrayLikeObject=hu,An.isBoolean=function(n){return true===n||false===n||yu(n)&&"[object Boolean]"==Ot(n)},An.isBuffer=af,An.isDate=lf,An.isElement=function(n){return yu(n)&&1===n.nodeType&&!xu(n)},An.isEmpty=function(n){
if(null==n)return true;if(su(n)&&(ff(n)||typeof n=="string"||typeof n.splice=="function"||af(n)||_f(n)||of(n)))return!n.length;var t=vo(n);if("[object Map]"==t||"[object Set]"==t)return!n.size;if(ze(n))return!Vt(n).length;for(var r in n)if(oi.call(n,r))return false;return true},An.isEqual=function(n,t){return Mt(n,t)},An.isEqualWith=function(n,t,r){var e=(r=typeof r=="function"?r:T)?r(n,t):T;return e===T?Mt(n,t,T,r):!!e},An.isError=pu,An.isFinite=function(n){return typeof n=="number"&&Wi(n)},An.isFunction=_u,
An.isInteger=vu,An.isLength=gu,An.isMap=sf,An.isMatch=function(n,t){return n===t||$t(n,t,xe(t))},An.isMatchWith=function(n,t,r){return r=typeof r=="function"?r:T,$t(n,t,xe(t),r)},An.isNaN=function(n){return bu(n)&&n!=+n},An.isNative=function(n){if(go(n))throw new Hu("Unsupported core-js use. Try https://npms.io/search?q=ponyfill.");return Ft(n)},An.isNil=function(n){return null==n},An.isNull=function(n){return null===n},An.isNumber=bu,An.isObject=du,An.isObjectLike=yu,An.isPlainObject=xu,An.isRegExp=hf,
An.isSafeInteger=function(n){return vu(n)&&-9007199254740991<=n&&9007199254740991>=n},An.isSet=pf,An.isString=ju,An.isSymbol=wu,An.isTypedArray=_f,An.isUndefined=function(n){return n===T},An.isWeakMap=function(n){return yu(n)&&"[object WeakMap]"==vo(n)},An.isWeakSet=function(n){return yu(n)&&"[object WeakSet]"==Ot(n)},An.join=function(n,t){return null==n?"":Bi.call(n,t)},An.kebabCase=Lf,An.last=Ve,An.lastIndexOf=function(n,t,r){var e=null==n?0:n.length;if(!e)return-1;var u=e;if(r!==T&&(u=Eu(r),u=0>u?Ui(e+u,0):Ci(u,e-1)),
t===t){for(r=u+1;r--&&n[r]!==t;);n=r}else n=_(n,d,u,true);return n},An.lowerCase=Uf,An.lowerFirst=Cf,An.lt=vf,An.lte=gf,An.max=function(n){return n&&n.length?xt(n,$u,It):T},An.maxBy=function(n,t){return n&&n.length?xt(n,ye(t,2),It):T},An.mean=function(n){return y(n,$u)},An.meanBy=function(n,t){return y(n,ye(t,2))},An.min=function(n){return n&&n.length?xt(n,$u,Kt):T},An.minBy=function(n,t){return n&&n.length?xt(n,ye(t,2),Kt):T},An.stubArray=qu,An.stubFalse=Vu,An.stubObject=function(){return{}},An.stubString=function(){
return""},An.stubTrue=function(){return true},An.multiply=rc,An.nth=function(n,t){return n&&n.length?Qt(n,Eu(t)):T},An.noConflict=function(){return $n._===this&&($n._=si),this},An.noop=Pu,An.now=Go,An.pad=function(n,t,r){n=Iu(n);var e=(t=Eu(t))?D(n):0;return!t||e>=t?n:(t=(t-e)/2,ne(Ii(t),r)+n+ne(Oi(t),r))},An.padEnd=function(n,t,r){n=Iu(n);var e=(t=Eu(t))?D(n):0;return t&&e<t?n+ne(t-e,r):n},An.padStart=function(n,t,r){n=Iu(n);var e=(t=Eu(t))?D(n):0;return t&&e<t?ne(t-e,r)+n:n},An.parseInt=function(n,t,r){
return r||null==t?t=0:t&&(t=+t),Mi(Iu(n).replace(on,""),t||0)},An.random=function(n,t,r){if(r&&typeof r!="boolean"&&Oe(n,t,r)&&(t=r=T),r===T&&(typeof t=="boolean"?(r=t,t=T):typeof n=="boolean"&&(r=n,n=T)),n===T&&t===T?(n=0,t=1):(n=Au(n),t===T?(t=n,n=0):t=Au(t)),n>t){var e=n;n=t,t=e}return r||n%1||t%1?(r=Ti(),Ci(n+r*(t-n+Cn("1e-"+((r+"").length-1))),t)):ir(n,t)},An.reduce=function(n,t,r){var e=ff(n)?l:j,u=3>arguments.length;return e(n,ye(t,4),r,u,uo)},An.reduceRight=function(n,t,r){var e=ff(n)?s:j,u=3>arguments.length;
return e(n,ye(t,4),r,u,io)},An.repeat=function(n,t,r){return t=(r?Oe(n,t,r):t===T)?1:Eu(t),or(Iu(n),t)},An.replace=function(){var n=arguments,t=Iu(n[0]);return 3>n.length?t:t.replace(n[1],n[2])},An.result=function(n,t,r){t=Sr(t,n);var e=-1,u=t.length;for(u||(u=1,n=T);++e<u;){var i=null==n?T:n[Me(t[e])];i===T&&(e=u,i=r),n=_u(i)?i.call(n):i}return n},An.round=ec,An.runInContext=x,An.sample=function(n){return(ff(n)?Qn:cr)(n)},An.size=function(n){if(null==n)return 0;if(su(n))return ju(n)?D(n):n.length;
var t=vo(n);return"[object Map]"==t||"[object Set]"==t?n.size:Vt(n).length},An.snakeCase=Df,An.some=function(n,t,r){var e=ff(n)?h:pr;return r&&Oe(n,t,r)&&(t=T),e(n,ye(t,3))},An.sortedIndex=function(n,t){return _r(n,t)},An.sortedIndexBy=function(n,t,r){return vr(n,t,ye(r,2))},An.sortedIndexOf=function(n,t){var r=null==n?0:n.length;if(r){var e=_r(n,t);if(e<r&&lu(n[e],t))return e}return-1},An.sortedLastIndex=function(n,t){return _r(n,t,true)},An.sortedLastIndexBy=function(n,t,r){return vr(n,t,ye(r,2),true);
},An.sortedLastIndexOf=function(n,t){if(null==n?0:n.length){var r=_r(n,t,true)-1;if(lu(n[r],t))return r}return-1},An.startCase=Mf,An.startsWith=function(n,t,r){return n=Iu(n),r=null==r?0:pt(Eu(r),0,n.length),t=yr(t),n.slice(r,r+t.length)==t},An.subtract=uc,An.sum=function(n){return n&&n.length?m(n,$u):0},An.sumBy=function(n,t){return n&&n.length?m(n,ye(t,2)):0},An.template=function(n,t,r){var e=An.templateSettings;r&&Oe(n,t,r)&&(t=T),n=Iu(n),t=bf({},t,e,ce),r=bf({},t.imports,e.imports,ce);var u,i,o=Wu(r),f=S(r,o),c=0;
r=t.interpolate||jn;var a="__p+='";r=Xu((t.escape||jn).source+"|"+r.source+"|"+(r===Q?pn:jn).source+"|"+(t.evaluate||jn).source+"|$","g");var l=oi.call(t,"sourceURL")?"//# sourceURL="+(t.sourceURL+"").replace(/[\r\n]/g," ")+"\n":"";if(n.replace(r,function(t,r,e,o,f,l){return e||(e=o),a+=n.slice(c,l).replace(wn,z),r&&(u=true,a+="'+__e("+r+")+'"),f&&(i=true,a+="';"+f+";\n__p+='"),e&&(a+="'+((__t=("+e+"))==null?'':__t)+'"),c=l+t.length,t}),a+="';",(t=oi.call(t,"variable")&&t.variable)||(a="with(obj){"+a+"}"),
a=(i?a.replace(P,""):a).replace(Z,"$1").replace(q,"$1;"),a="function("+(t||"obj")+"){"+(t?"":"obj||(obj={});")+"var __t,__p=''"+(u?",__e=_.escape":"")+(i?",__j=Array.prototype.join;function print(){__p+=__j.call(arguments,'')}":";")+a+"return __p}",t=Ff(function(){return Ju(o,l+"return "+a).apply(T,f)}),t.source=a,pu(t))throw t;return t},An.times=function(n,t){if(n=Eu(n),1>n||9007199254740991<n)return[];var r=4294967295,e=Ci(n,4294967295);for(t=ye(t),n-=4294967295,e=A(e,t);++r<n;)t(r);return e},An.toFinite=Au,
An.toInteger=Eu,An.toLength=ku,An.toLower=function(n){return Iu(n).toLowerCase()},An.toNumber=Su,An.toSafeInteger=function(n){return n?pt(Eu(n),-9007199254740991,9007199254740991):0===n?n:0},An.toString=Iu,An.toUpper=function(n){return Iu(n).toUpperCase()},An.trim=function(n,t,r){return(n=Iu(n))&&(r||t===T)?n.replace(un,""):n&&(t=yr(t))?(n=M(n),r=M(t),t=I(n,r),r=R(n,r)+1,Or(n,t,r).join("")):n},An.trimEnd=function(n,t,r){return(n=Iu(n))&&(r||t===T)?n.replace(fn,""):n&&(t=yr(t))?(n=M(n),t=R(n,M(t))+1,
Or(n,0,t).join("")):n},An.trimStart=function(n,t,r){return(n=Iu(n))&&(r||t===T)?n.replace(on,""):n&&(t=yr(t))?(n=M(n),t=I(n,M(t)),Or(n,t).join("")):n},An.truncate=function(n,t){var r=30,e="...";if(du(t))var u="separator"in t?t.separator:u,r="length"in t?Eu(t.length):r,e="omission"in t?yr(t.omission):e;n=Iu(n);var i=n.length;if(Rn.test(n))var o=M(n),i=o.length;if(r>=i)return n;if(i=r-D(e),1>i)return e;if(r=o?Or(o,0,i).join(""):n.slice(0,i),u===T)return r+e;if(o&&(i+=r.length-i),hf(u)){if(n.slice(i).search(u)){
var f=r;for(u.global||(u=Xu(u.source,Iu(_n.exec(u))+"g")),u.lastIndex=0;o=u.exec(f);)var c=o.index;r=r.slice(0,c===T?i:c)}}else n.indexOf(yr(u),i)!=i&&(u=r.lastIndexOf(u),-1<u&&(r=r.slice(0,u)));return r+e},An.unescape=function(n){return(n=Iu(n))&&G.test(n)?n.replace(V,tt):n},An.uniqueId=function(n){var t=++fi;return Iu(n)+t},An.upperCase=Tf,An.upperFirst=$f,An.each=nu,An.eachRight=tu,An.first=qe,Nu(An,function(){var n={};return mt(An,function(t,r){oi.call(An.prototype,r)||(n[r]=t)}),n}(),{chain:false
}),An.VERSION="4.17.15",r("bind bindKey curry curryRight partial partialRight".split(" "),function(n){An[n].placeholder=An}),r(["drop","take"],function(n,t){Un.prototype[n]=function(r){r=r===T?1:Ui(Eu(r),0);var e=this.__filtered__&&!t?new Un(this):this.clone();return e.__filtered__?e.__takeCount__=Ci(r,e.__takeCount__):e.__views__.push({size:Ci(r,4294967295),type:n+(0>e.__dir__?"Right":"")}),e},Un.prototype[n+"Right"]=function(t){return this.reverse()[n](t).reverse()}}),r(["filter","map","takeWhile"],function(n,t){
var r=t+1,e=1==r||3==r;Un.prototype[n]=function(n){var t=this.clone();return t.__iteratees__.push({iteratee:ye(n,3),type:r}),t.__filtered__=t.__filtered__||e,t}}),r(["head","last"],function(n,t){var r="take"+(t?"Right":"");Un.prototype[n]=function(){return this[r](1).value()[0]}}),r(["initial","tail"],function(n,t){var r="drop"+(t?"":"Right");Un.prototype[n]=function(){return this.__filtered__?new Un(this):this[r](1)}}),Un.prototype.compact=function(){return this.filter($u)},Un.prototype.find=function(n){
return this.filter(n).head()},Un.prototype.findLast=function(n){return this.reverse().find(n)},Un.prototype.invokeMap=fr(function(n,t){return typeof n=="function"?new Un(this):this.map(function(r){return Lt(r,n,t)})}),Un.prototype.reject=function(n){return this.filter(au(ye(n)))},Un.prototype.slice=function(n,t){n=Eu(n);var r=this;return r.__filtered__&&(0<n||0>t)?new Un(r):(0>n?r=r.takeRight(-n):n&&(r=r.drop(n)),t!==T&&(t=Eu(t),r=0>t?r.dropRight(-t):r.take(t-n)),r)},Un.prototype.takeRightWhile=function(n){
return this.reverse().takeWhile(n).reverse()},Un.prototype.toArray=function(){return this.take(4294967295)},mt(Un.prototype,function(n,t){var r=/^(?:filter|find|map|reject)|While$/.test(t),e=/^(?:head|last)$/.test(t),u=An[e?"take"+("last"==t?"Right":""):t],i=e||/^find/.test(t);u&&(An.prototype[t]=function(){function t(n){return n=u.apply(An,a([n],f)),e&&h?n[0]:n}var o=this.__wrapped__,f=e?[1]:arguments,c=o instanceof Un,l=f[0],s=c||ff(o);s&&r&&typeof l=="function"&&1!=l.length&&(c=s=false);var h=this.__chain__,p=!!this.__actions__.length,l=i&&!h,c=c&&!p;
return!i&&s?(o=c?o:new Un(this),o=n.apply(o,f),o.__actions__.push({func:Qe,args:[t],thisArg:T}),new On(o,h)):l&&c?n.apply(this,f):(o=this.thru(t),l?e?o.value()[0]:o.value():o)})}),r("pop push shift sort splice unshift".split(" "),function(n){var t=ri[n],r=/^(?:push|sort|unshift)$/.test(n)?"tap":"thru",e=/^(?:pop|shift)$/.test(n);An.prototype[n]=function(){var n=arguments;if(e&&!this.__chain__){var u=this.value();return t.apply(ff(u)?u:[],n)}return this[r](function(r){return t.apply(ff(r)?r:[],n)});
}}),mt(Un.prototype,function(n,t){var r=An[t];if(r){var e=r.name+"";oi.call(Gi,e)||(Gi[e]=[]),Gi[e].push({name:t,func:r})}}),Gi[Jr(T,2).name]=[{name:"wrapper",func:T}],Un.prototype.clone=function(){var n=new Un(this.__wrapped__);return n.__actions__=Ur(this.__actions__),n.__dir__=this.__dir__,n.__filtered__=this.__filtered__,n.__iteratees__=Ur(this.__iteratees__),n.__takeCount__=this.__takeCount__,n.__views__=Ur(this.__views__),n},Un.prototype.reverse=function(){if(this.__filtered__){var n=new Un(this);
n.__dir__=-1,n.__filtered__=true}else n=this.clone(),n.__dir__*=-1;return n},Un.prototype.value=function(){var n,t=this.__wrapped__.value(),r=this.__dir__,e=ff(t),u=0>r,i=e?t.length:0;n=i;for(var o=this.__views__,f=0,c=-1,a=o.length;++c<a;){var l=o[c],s=l.size;switch(l.type){case"drop":f+=s;break;case"dropRight":n-=s;break;case"take":n=Ci(n,f+s);break;case"takeRight":f=Ui(f,n-s)}}if(n={start:f,end:n},o=n.start,f=n.end,n=f-o,o=u?f:o-1,f=this.__iteratees__,c=f.length,a=0,l=Ci(n,this.__takeCount__),!e||!u&&i==n&&l==n)return wr(t,this.__actions__);
e=[];n:for(;n--&&a<l;){for(o+=r,u=-1,i=t[o];++u<c;){var h=f[u],s=h.type,h=(0,h.iteratee)(i);if(2==s)i=h;else if(!h){if(1==s)continue n;break n}}e[a++]=i}return e},An.prototype.at=To,An.prototype.chain=function(){return Ye(this)},An.prototype.commit=function(){return new On(this.value(),this.__chain__)},An.prototype.next=function(){this.__values__===T&&(this.__values__=mu(this.value()));var n=this.__index__>=this.__values__.length;return{done:n,value:n?T:this.__values__[this.__index__++]}},An.prototype.plant=function(n){
for(var t,r=this;r instanceof En;){var e=Fe(r);e.__index__=0,e.__values__=T,t?u.__wrapped__=e:t=e;var u=e,r=r.__wrapped__}return u.__wrapped__=n,t},An.prototype.reverse=function(){var n=this.__wrapped__;return n instanceof Un?(this.__actions__.length&&(n=new Un(this)),n=n.reverse(),n.__actions__.push({func:Qe,args:[Ge],thisArg:T}),new On(n,this.__chain__)):this.thru(Ge)},An.prototype.toJSON=An.prototype.valueOf=An.prototype.value=function(){return wr(this.__wrapped__,this.__actions__)},An.prototype.first=An.prototype.head,
wi&&(An.prototype[wi]=Xe),An}();typeof define=="function"&&typeof define.amd=="object"&&define.amd?($n._=rt, define(function(){return rt})):Nn?((Nn.exports=rt)._=rt,Fn._=rt):$n._=rt}).call(this);

//third_party/javascript/angular/v1_6/angular.min.js
/*
 AngularJS v1.6.4-local+sha.617b36117
 (c) 2010-2018 Google, Inc. http://angularjs.org
 License: MIT
*/
'use strict';(function(D){'use strict';function te(a){if(G(a))v(a.objectMaxDepth)&&(Uc.objectMaxDepth=ac(a.objectMaxDepth)?a.objectMaxDepth:NaN);else return Uc}function ac(a){return ca(a)&&0<a}function K(a){return function(){var b=arguments[0];var d="["+(a?a+":":"")+b+"] http://errors.angularjs.org/1.6.4-local+sha.617b36117/"+(a?a+"/":"")+b;for(b=1;b<arguments.length;b++){d=d+(1==b?"?":"&")+"p"+(b-1)+"=";var c=encodeURIComponent;var e=arguments[b];e="function"==typeof e?e.toString().replace(/ \{[\s\S]*$/,""):
"undefined"==typeof e?"undefined":"string"!=typeof e?JSON.stringify(e):e;d+=c(e)}return Error(d)}}function pa(a){if(null==a||cb(a))return!1;if(J(a)||I(a)||x&&a instanceof x)return!0;var b="length"in Object(a)&&a.length;return ca(b)&&(0<=b&&(b-1 in a||a instanceof Array)||"function"===typeof a.item)}function p(a,b,d){var c;if(a)if(A(a))for(g in a)"prototype"!==g&&"length"!==g&&"name"!==g&&a.hasOwnProperty(g)&&b.call(d,a[g],g,a);else if(J(a)||pa(a)){var e="object"!==typeof a;var g=0;for(c=a.length;g<
c;g++)(e||g in a)&&b.call(d,a[g],g,a)}else if(a.forEach&&a.forEach!==p)a.forEach(b,d,a);else if(Vc(a))for(g in a)b.call(d,a[g],g,a);else if("function"===typeof a.hasOwnProperty)for(g in a)a.hasOwnProperty(g)&&b.call(d,a[g],g,a);else for(g in a)ua.call(a,g)&&b.call(d,a[g],g,a);return a}function Wc(a,b,d){for(var c=Object.keys(a).sort(),e=0;e<c.length;e++)b.call(d,a[c[e]],c[e]);return c}function bc(a){return function(b,d){a(d,b)}}function ue(){return++wb}function cc(a,b,d){for(var c=a.$$hashKey,e=0,
g=b.length;e<g;++e){var f=b[e];if(G(f)||A(f))for(var h=Object.keys(f),k=0,l=h.length;k<l;k++){var m=h[k],n=f[m];d&&G(n)?ia(n)?a[m]=new Date(n.valueOf()):xb(n)?a[m]=new RegExp(n):n.nodeName?a[m]=n.cloneNode(!0):dc(n)?a[m]=n.clone():(G(a[m])||(a[m]=J(n)?[]:{}),cc(a[m],[n],!0)):a[m]=n}}c?a.$$hashKey=c:delete a.$$hashKey;return a}function P(a){return cc(a,ya.call(arguments,1),!1)}function ve(a){return cc(a,ya.call(arguments,1),!0)}function ec(a,b){return P(Object.create(a),b)}function H(){}function yb(a){return a}
function la(a){return function(){return a}}function fc(a){return A(a.toString)&&a.toString!==ma}function z(a){return"undefined"===typeof a}function v(a){return"undefined"!==typeof a}function G(a){return null!==a&&"object"===typeof a}function Vc(a){return null!==a&&"object"===typeof a&&!Xc(a)}function I(a){return"string"===typeof a}function ca(a){return"number"===typeof a}function ia(a){return"[object Date]"===ma.call(a)}function gc(a){switch(ma.call(a)){case "[object Error]":return!0;case "[object Exception]":return!0;
case "[object DOMException]":return!0;default:return a instanceof Error}}function A(a){return"function"===typeof a}function xb(a){return"[object RegExp]"===ma.call(a)}function cb(a){return a&&a.window===a}function db(a){return a&&a.$evalAsync&&a.$watch}function Ra(a){return"boolean"===typeof a}function we(a){return a&&ca(a.length)&&xe.test(ma.call(a))}function dc(a){return!(!a||!(a.nodeName||a.prop&&a.attr&&a.find))}function ye(a){var b={};a=a.split(",");var d;for(d=0;d<a.length;d++)b[a[d]]=!0;return b}
function za(a){return N(a.nodeName||a[0]&&a[0].nodeName)}function eb(a,b){b=a.indexOf(b);0<=b&&a.splice(b,1);return b}function Ga(a,b,d){function c(a,b,c){c--;if(0>c)return"...";var d=b.$$hashKey;if(J(a)){var f=0;for(var g=a.length;f<g;f++)b.push(e(a[f],c))}else if(Vc(a))for(f in a)b[f]=e(a[f],c);else if(a&&"function"===typeof a.hasOwnProperty)for(f in a)a.hasOwnProperty(f)&&(b[f]=e(a[f],c));else for(f in a)ua.call(a,f)&&(b[f]=e(a[f],c));d?b.$$hashKey=d:delete b.$$hashKey;return b}function e(a,b){if(!G(a))return a;
var d=f.indexOf(a);if(-1!==d)return h[d];if(cb(a)||db(a))throw qa("cpws");d=!1;var e=g(a);void 0===e&&(e=J(a)?[]:Object.create(Xc(a)),d=!0);f.push(a);h.push(e);return d?c(a,e,b):e}function g(a){switch(ma.call(a)){case "[object Int8Array]":case "[object Int16Array]":case "[object Int32Array]":case "[object Float32Array]":case "[object Float64Array]":case "[object Uint8Array]":case "[object Uint8ClampedArray]":case "[object Uint16Array]":case "[object Uint32Array]":return new a.constructor(e(a.buffer),
a.byteOffset,a.length);case "[object ArrayBuffer]":if(!a.slice){var b=new ArrayBuffer(a.byteLength);(new Uint8Array(b)).set(new Uint8Array(a));return b}return a.slice(0);case "[object Boolean]":case "[object Number]":case "[object String]":case "[object Date]":return new a.constructor(a.valueOf());case "[object RegExp]":return b=new RegExp(a.source,a.toString().match(/[^/]*$/)[0]),b.lastIndex=a.lastIndex,b;case "[object Blob]":return new a.constructor([a],{type:a.type})}if(A(a.cloneNode))return a.cloneNode(!0)}
var f=[],h=[];d=ac(d)?d:NaN;if(b){if(we(b)||"[object ArrayBuffer]"===ma.call(b))throw qa("cpta");if(a===b)throw qa("cpi");J(b)?b.length=0:p(b,function(a,c){"$$hashKey"!==c&&delete b[c]});f.push(a);h.push(b);return c(a,b,d)}return e(a,d)}function hc(a,b){return a===b||a!==a&&b!==b}function va(a,b){if(a===b)return!0;if(null===a||null===b)return!1;if(a!==a&&b!==b)return!0;var d=typeof a,c;if(d===typeof b&&"object"===d)if(J(a)){if(!J(b))return!1;if((d=a.length)===b.length){for(c=0;c<d;c++)if(!va(a[c],
b[c]))return!1;return!0}}else{if(ia(a))return ia(b)?hc(a.getTime(),b.getTime()):!1;if(xb(a))return xb(b)?a.toString()===b.toString():!1;if(db(a)||db(b)||cb(a)||cb(b)||J(b)||ia(b)||xb(b))return!1;d=V();for(c in a)if("$"!==c.charAt(0)&&!A(a[c])){if(!va(a[c],b[c]))return!1;d[c]=!0}for(c in b)if(!(c in d)&&"$"!==c.charAt(0)&&v(b[c])&&!A(b[c]))return!1;return!0}return!1}function fb(a,b,d){return a.concat(ya.call(b,d))}function Xa(a,b){var d=2<arguments.length?ya.call(arguments,2):[];return!A(b)||b instanceof
RegExp?b:d.length?function(){return arguments.length?b.apply(a,fb(d,arguments,0)):b.apply(a,d)}:function(){return arguments.length?b.apply(a,arguments):b.call(a)}}function Yc(a,b){var d=b;"string"===typeof a&&"$"===a.charAt(0)&&"$"===a.charAt(1)?d=void 0:cb(b)?d="$WINDOW":b&&D.document===b?d="$DOCUMENT":db(b)&&(d="$SCOPE");return d}function gb(a,b){if(!z(a))return ca(b)||(b=b?2:null),JSON.stringify(a,Yc,b)}function Zc(a){return I(a)?JSON.parse(a):a}function ic(a,b){a=a.replace(ze,"");a=Date.parse("Jan 01, 1970 00:00:00 "+
a)/6E4;return aa(a)?b:a}function $c(a,b){a=new Date(a.getTime());a.setMinutes(a.getMinutes()+b);return a}function jc(a,b,d){d=d?-1:1;var c=a.getTimezoneOffset();b=ic(b,c);return $c(a,d*(b-c))}function Aa(a){a=x(a).clone().empty();var b=x("<div></div>").append(a).html();try{return a[0].nodeType===Sa?N(b):b.match(/^(<[^>]+>)/)[1].replace(/^<([\w-]+)/,function(a,b){return"<"+N(b)})}catch(d){return N(b)}}function ad(a){try{return decodeURIComponent(a)}catch(b){}}function kc(a){var b={};p((a||"").split("&"),
function(a){if(a){var c=a=a.replace(/\+/g,"%20");var d=a.indexOf("=");if(-1!==d){c=a.substring(0,d);var g=a.substring(d+1)}c=ad(c);v(c)&&(g=v(g)?ad(g):!0,ua.call(b,c)?J(b[c])?b[c].push(g):b[c]=[b[c],g]:b[c]=g)}});return b}function lc(a){var b=[];p(a,function(a,c){J(a)?p(a,function(a){b.push(na(c,!0)+(!0===a?"":"="+na(a,!0)))}):b.push(na(c,!0)+(!0===a?"":"="+na(a,!0)))});return b.length?b.join("&"):""}function hb(a){return na(a,!0).replace(/%26/gi,"&").replace(/%3D/gi,"=").replace(/%2B/gi,"+")}function na(a,
b){return encodeURIComponent(a).replace(/%40/gi,"@").replace(/%3A/gi,":").replace(/%24/g,"$").replace(/%2C/gi,",").replace(/%3B/gi,";").replace(/%20/g,b?"%20":"+")}function Ae(a,b){var d,c=Ha.length;for(d=0;d<c;++d){var e=Ha[d]+b;if(I(e=a.getAttribute(e)))return e}return null}function Be(a,b){var d,c,e={};p(Ha,function(b){b+="app";!d&&a.hasAttribute&&a.hasAttribute(b)&&(d=a,c=a.getAttribute(b))});p(Ha,function(b){b+="app";var e;!d&&(e=a.querySelector("["+b.replace(":","\\:")+"]"))&&(d=e,c=e.getAttribute(b))});
d&&(Ce?(e.strictDi=null!==Ae(d,"strict-di"),b(d,c?[c]:[],e)):D.console.error("AngularJS: disabling automatic bootstrap. <script> protocol indicates an extension, document.location.href does not match."))}function bd(a,b,d){G(d)||(d={});d=P({strictDi:!1},d);var c=function(){a=x(a);if(a.injector()){var c=a[0]===D.document?"document":Aa(a);throw qa("btstrpd",c.replace(/</,"&lt;").replace(/>/,"&gt;"));}b=b||[];b.unshift(["$provide",function(b){b.value("$rootElement",a)}]);d.debugInfoEnabled&&b.push(["$compileProvider",
function(a){a.debugInfoEnabled(!0)}]);b.unshift("ng");c=ib(b,d.strictDi);c.invoke(["$rootScope","$rootElement","$compile","$injector",function(a,b,c,d){a.$apply(function(){b.data("$injector",d);c(b)(a)})}]);return c},e=/^NG_ENABLE_DEBUG_INFO!/,g=/^NG_DEFER_BOOTSTRAP!/;D&&e.test(D.name)&&(d.debugInfoEnabled=!0,D.name=D.name.replace(e,""));if(D&&!g.test(D.name))return c();D.name=D.name.replace(g,"");ja.resumeBootstrap=function(a){p(a,function(a){b.push(a)});return c()};A(ja.resumeDeferredBootstrap)&&
ja.resumeDeferredBootstrap()}function De(){D.name="NG_ENABLE_DEBUG_INFO!"+D.name;D.location.reload()}function Ee(a){a=ja.element(a).injector();if(!a)throw qa("test");return a.get("$$testability")}function cd(a,b){b=b||"_";return a.replace(Fe,function(a,c){return(c?b:"")+a.toLowerCase()})}function jb(a,b,d){if(!a)throw qa("areq",b||"?",d||"required");return a}function zb(a,b,d){d&&J(a)&&(a=a[a.length-1]);jb(A(a),b,"not a function, got "+(a&&"object"===typeof a?a.constructor.name||"Object":typeof a));
return a}function Ia(a,b){if("hasOwnProperty"===a)throw qa("badname",b);}function dd(a,b,d){if(!b)return a;b=b.split(".");for(var c,e=a,g=b.length,f=0;f<g;f++)c=b[f],a&&(a=(e=a)[c]);return!d&&A(a)?Xa(e,a):a}function Ab(a){for(var b=a[0],d=a[a.length-1],c,e=1;b!==d&&(b=b.nextSibling);e++)if(c||a[e]!==b)c||(c=x(ya.call(a,0,e))),c.push(b);return c||a}function V(){return Object.create(null)}function mc(a){if(null==a)return"";switch(typeof a){case "string":break;case "number":a=""+a;break;default:a=!fc(a)||
J(a)||ia(a)?gb(a):a.toString()}return a}function Ge(a){function b(a,b,c){return a[b]||(a[b]=c())}var d=K("$injector"),c=K("ng");a=b(a,"angular",Object);a.$$minErr=a.$$minErr||K;return b(a,"module",function(){var a={};return function(e,f,h){var g={};if("hasOwnProperty"===e)throw c("badname","module");f&&a.hasOwnProperty(e)&&(a[e]=null);return b(a,e,function(){function a(a,b,c,d){d||(d=k);return function(){d[c||"push"]([a,b,arguments]);return u}}function b(a,b,c){c||(c=k);return function(d,f){f&&A(f)&&
(f.$$moduleName=e);c.push([a,b,arguments]);return u}}if(!f)throw d("nomod",e);var k=[],t=[],E=[],B=a("$injector","invoke","push",t),u={_invokeQueue:k,_configBlocks:t,_runBlocks:E,info:function(a){if(v(a)){if(!G(a))throw c("aobj","value");g=a;return this}return g},requires:f,name:e,provider:b("$provide","provider"),factory:b("$provide","factory"),service:b("$provide","service"),value:a("$provide","value"),constant:a("$provide","constant","unshift"),decorator:b("$provide","decorator",t),animation:b("$animateProvider",
"register"),filter:b("$filterProvider","register"),controller:b("$controllerProvider","register"),directive:b("$compileProvider","directive"),component:b("$compileProvider","component"),config:B,run:function(a){E.push(a);return this}};h&&B(h);return u})}})}function ra(a,b){if(J(a)){b=b||[];for(var d=0,c=a.length;d<c;d++)b[d]=a[d]}else if(G(a))for(d in b=b||{},a)if("$"!==d.charAt(0)||"$"!==d.charAt(1))b[d]=a[d];return b||a}function He(a,b){var d=[];ac(b)&&(a=ja.copy(a,null,b));return JSON.stringify(a,
function(a,b){b=Yc(a,b);if(G(b)){if(0<=d.indexOf(b))return"...";d.push(b)}return b})}function Ba(a,b){return b.toUpperCase()}function nc(a){a=a.nodeType;return 1===a||!a||9===a}function ed(a,b){var d=b.createDocumentFragment(),c=[];if(oc.test(a)){b=d.appendChild(b.createElement("div"));var e=(Ie.exec(a)||["",""])[1].toLowerCase();e=oa[e]||oa._default;b.innerHTML=e[1]+a.replace(Je,"<$1></$2>")+e[2];for(a=e[0];a--;)b=b.lastChild;c=fb(c,b.childNodes);b=d.firstChild;b.textContent=""}else c.push(b.createTextNode(a));
d.textContent="";d.innerHTML="";p(c,function(a){d.appendChild(a)});return d}function Y(a){if(a instanceof Y)return a;if(I(a)){a=T(a);var b=!0}if(!(this instanceof Y)){if(b&&"<"!==a.charAt(0))throw pc("nosel");return new Y(a)}if(b){b=D.document;var d;a=(d=Ke.exec(a))?[b.createElement(d[1])]:(d=ed(a,b))?d.childNodes:[];qc(this,a)}else A(a)?fd(a):qc(this,a)}function rc(a){return a.cloneNode(!0)}function Bb(a,b){!b&&nc(a)&&x.cleanData([a]);a.querySelectorAll&&x.cleanData(a.querySelectorAll("*"))}function gd(a,
b,d,c){if(v(c))throw pc("offargs");var e=(c=Cb(a))&&c.events,g=c&&c.handle;if(g)if(b){var f=function(b){var c=e[b];v(d)&&eb(c||[],d);v(d)&&c&&0<c.length||(a.removeEventListener(b,g),delete e[b])};p(b.split(" "),function(a){f(a);Db[a]&&f(Db[a])})}else for(b in e)"$destroy"!==b&&a.removeEventListener(b,g),delete e[b]}function sc(a,b){var d=a.ng339,c=d&&lb[d];c&&(b?delete c.data[b]:(c.handle&&(c.events.$destroy&&c.handle({},"$destroy"),gd(a)),delete lb[d],a.ng339=void 0))}function Cb(a,b){var d=a.ng339;
d=d&&lb[d];b&&!d&&(a.ng339=d=++Le,d=lb[d]={events:{},data:{},handle:void 0});return d}function tc(a,b,d){if(nc(a)){var c,e=v(d),g=!e&&b&&!G(b),f=!b;a=(a=Cb(a,!g))&&a.data;if(e)a[b.replace(Eb,Ba)]=d;else{if(f)return a;if(g)return a&&a[b.replace(Eb,Ba)];for(c in b)a[c.replace(Eb,Ba)]=b[c]}}}function Fb(a,b){return a.getAttribute?-1<(" "+(a.getAttribute("class")||"")+" ").replace(/[\n\t]/g," ").indexOf(" "+b+" "):!1}function Gb(a,b){if(b&&a.setAttribute){var d=(" "+(a.getAttribute("class")||"")+" ").replace(/[\n\t]/g,
" "),c=d;p(b.split(" "),function(a){a=T(a);c=c.replace(" "+a+" "," ")});c!==d&&a.setAttribute("class",T(c))}}function Hb(a,b){if(b&&a.setAttribute){var d=(" "+(a.getAttribute("class")||"")+" ").replace(/[\n\t]/g," "),c=d;p(b.split(" "),function(a){a=T(a);-1===c.indexOf(" "+a+" ")&&(c+=a+" ")});c!==d&&a.setAttribute("class",T(c))}}function qc(a,b){if(b)if(b.nodeType)a[a.length++]=b;else{var d=b.length;if("number"===typeof d&&b.window!==b){if(d)for(var c=0;c<d;c++)a[a.length++]=b[c]}else a[a.length++]=
b}}function hd(a,b){return Ib(a,"$"+(b||"ngController")+"Controller")}function Ib(a,b,d){9===a.nodeType&&(a=a.documentElement);for(b=J(b)?b:[b];a;){for(var c=0,e=b.length;c<e;c++)if(v(d=x.data(a,b[c])))return d;a=a.parentNode||11===a.nodeType&&a.host}}function id(a){for(Bb(a,!0);a.firstChild;)a.removeChild(a.firstChild)}function Jb(a,b){b||Bb(a);(b=a.parentNode)&&b.removeChild(a)}function Me(a,b){b=b||D;if("complete"===b.document.readyState)b.setTimeout(a);else x(b).on("load",a)}function fd(a){function b(){D.document.removeEventListener("DOMContentLoaded",
b);D.removeEventListener("load",b);a()}"complete"===D.document.readyState?D.setTimeout(a):(D.document.addEventListener("DOMContentLoaded",b),D.addEventListener("load",b))}function jd(a,b){return(b=Kb[b.toLowerCase()])&&kd[za(a)]&&b}function Ne(a,b){var d=function(c,d){c.isDefaultPrevented=function(){return c.defaultPrevented};var e=(d=b[d||c.type])?d.length:0;if(e){if(z(c.immediatePropagationStopped)){var f=c.stopImmediatePropagation;c.stopImmediatePropagation=function(){c.immediatePropagationStopped=
!0;c.stopPropagation&&c.stopPropagation();f&&f.call(c)}}c.isImmediatePropagationStopped=function(){return!0===c.immediatePropagationStopped};var h=d.specialHandlerWrapper||Oe;1<e&&(d=ra(d));for(var k=0;k<e;k++)c.isImmediatePropagationStopped()||h(a,c,d[k])}};d.elem=a;return d}function Oe(a,b,d){d.call(a,b)}function Pe(a,b,d){var c=b.relatedTarget;c&&(c===a||Qe.call(a,c))||d.call(a,b)}function Re(){this.$get=function(){return P(Y,{hasClass:function(a,b){a.attr&&(a=a[0]);return Fb(a,b)},addClass:function(a,
b){a.attr&&(a=a[0]);return Hb(a,b)},removeClass:function(a,b){a.attr&&(a=a[0]);return Gb(a,b)}})}}function Ja(a,b){var d=a&&a.$$hashKey;if(d)return"function"===typeof d&&(d=a.$$hashKey()),d;d=typeof a;return d="function"===d||"object"===d&&null!==a?a.$$hashKey=d+":"+(b||ue)():d+":"+a}function ld(){this._keys=[];this._values=[];this._lastKey=NaN;this._lastIndex=-1}function md(a){a=Function.prototype.toString.call(a).replace(Se,"");return a.match(Te)||a.match(Ue)}function Ve(a){return(a=md(a))?"function("+
(a[1]||"").replace(/[\s\r\n]+/," ")+")":"fn"}function ib(a,b){function d(a){return function(b,c){if(G(b))p(b,bc(a));else return a(b,c)}}function c(a,b){Ia(a,"service");if(A(b)||J(b))b=t.instantiate(b);if(!b.$get)throw Ca("pget",a);return n[a+"Provider"]=b}function e(a,b){return function(){var c=u.invoke(b,this);if(z(c))throw Ca("undef",a);return c}}function g(a,b,d){return c(a,{$get:!1!==d?e(a,b):b})}function f(a){jb(z(a)||J(a),"modulesToLoad","not an array");var b=[],c;p(a,function(a){function d(a){var b;
var c=0;for(b=a.length;c<b;c++){var d=a[c],e=t.get(d[0]);e[d[1]].apply(e,d[2])}}if(!m.get(a)){m.set(a,!0);try{I(a)?(c=uc(a),u.modules[a]=c,b=b.concat(f(c.requires)).concat(c._runBlocks),d(c._invokeQueue),d(c._configBlocks)):A(a)?b.push(t.invoke(a)):J(a)?b.push(t.invoke(a)):zb(a,"module")}catch(C){throw J(a)&&(a=a[a.length-1]),C.message&&C.stack&&-1===C.stack.indexOf(C.message)&&(C=C.message+"\n"+C.stack),Ca("modulerr",a,C.stack||C.message||C);}}});return b}function h(a,c){function d(b,d){if(a.hasOwnProperty(b)){if(a[b]===
k)throw Ca("cdep",b+" <- "+l.join(" <- "));return a[b]}try{return l.unshift(b),a[b]=k,a[b]=c(b,d),a[b]}catch(w){throw a[b]===k&&delete a[b],w;}finally{l.shift()}}function e(a,c,e){var f=[];a=ib.$$annotate(a,b,e);for(var g=0,h=a.length;g<h;g++){var k=a[g];if("string"!==typeof k)throw Ca("itkn",k);f.push(c&&c.hasOwnProperty(k)?c[k]:d(k,e))}return f}return{invoke:function(a,b,c,d){"string"===typeof c&&(d=c,c=null);c=e(a,c,d);J(a)&&(a=a[a.length-1]);d=a;if(La||"function"!==typeof d)d=!1;else{var f=d.$$ngIsClass;
Ra(f)||(f=d.$$ngIsClass=/^(?:class\b|constructor\()/.test(Function.prototype.toString.call(d)));d=f}return d?(c.unshift(null),new (Function.prototype.bind.apply(a,c))):a.apply(b,c)},instantiate:function(a,b,c){var d=J(a)?a[a.length-1]:a;a=e(a,b,c);a.unshift(null);return new (Function.prototype.bind.apply(d,a))},get:d,annotate:ib.$$annotate,has:function(b){return n.hasOwnProperty(b+"Provider")||a.hasOwnProperty(b)}}}b=!0===b;var k={},l=[],m=new Lb,n={$provide:{provider:d(c),factory:d(g),service:d(function(a,
b){return g(a,["$injector",function(a){return a.instantiate(b)}])}),value:d(function(a,b){return g(a,la(b),!1)}),constant:d(function(a,b){Ia(a,"constant");n[a]=b;E[a]=b}),decorator:function(a,b){var c=t.get(a+"Provider"),d=c.$get;c.$get=function(){var a=u.invoke(d,c);return u.invoke(b,null,{$delegate:a})}}}},t=n.$injector=h(n,function(a,b){ja.isString(b)&&l.push(b);throw Ca("unpr",l.join(" <- "));}),E={},B=h(E,function(a,b){b=t.get(a+"Provider",b);return u.invoke(b.$get,b,void 0,a)}),u=B;n.$injectorProvider=
{$get:la(B)};u.modules=t.modules=V();a=f(a);u=B.get("$injector");u.strictDi=b;p(a,function(a){a&&u.invoke(a)});u.loadNewModules=function(a){p(f(a),function(a){a&&u.invoke(a)})};return u}function We(){var a=!0;this.disableAutoScrolling=function(){a=!1};this.$get=["$window","$location","$rootScope",function(b,d,c){function e(a){var b=null;Array.prototype.some.call(a,function(a){if("a"===za(a))return b=a,!0});return b}function g(a){if(a){a.scrollIntoView();var c=f.yOffset;A(c)?c=c():dc(c)?(c=c[0],c=
"fixed"!==b.getComputedStyle(c).position?0:c.getBoundingClientRect().bottom):ca(c)||(c=0);c&&(a=a.getBoundingClientRect().top,b.scrollBy(0,a-c))}else b.scrollTo(0,0)}function f(a){a=I(a)?a:ca(a)?a.toString():d.hash();var b;a?(b=h.getElementById(a))?g(b):(b=e(h.getElementsByName(a)))?g(b):"top"===a&&g(null):g(null)}var h=b.document;a&&c.$watch(function(){return d.hash()},function(a,b){a===b&&""===a||Me(function(){c.$evalAsync(f)})});return f}]}function mb(a,b){if(!a&&!b)return"";if(!a)return b;if(!b)return a;
J(a)&&(a=a.join(" "));J(b)&&(b=b.join(" "));return a+" "+b}function Xe(a){I(a)&&(a=a.split(" "));var b=V();p(a,function(a){a.length&&(b[a]=!0)});return b}function sa(a){return G(a)?a:{}}function Ye(a,b,d,c){function e(a){try{a.apply(null,ya.call(arguments,1))}finally{if(B--,0===B)for(;u.length;)try{u.pop()()}catch(L){d.error(L)}}}function g(){y=null;h()}function f(){F=C();F=z(F)?null:F;va(F,U)&&(F=U);r=U=F}function h(){var a=r;f();if(O!==k.url()||a!==F)O=k.url(),r=F,p(w,function(a){a(k.url(),F)})}
var k=this,l=a.location,m=a.history,n=a.setTimeout,t=a.clearTimeout,E={};k.isMock=!1;var B=0,u=[];k.$$completeOutstandingRequest=e;k.$$incOutstandingRequestCount=function(){B++};k.notifyWhenNoOutstandingRequests=function(a){0===B?a():u.push(a)};var F,r,O=l.href,kb=b.find("base"),y=null,C=c.history?function(){try{return m.state}catch(Ka){}}:H;f();k.url=function(b,d,e){z(e)&&(e=null);l!==a.location&&(l=a.location);m!==a.history&&(m=a.history);if(b){var g=r===e;if(O===b&&(!c.history||g))return k;var h=
O&&ta(O)===ta(b);O=b;r=e;!c.history||h&&g?(h||(y=b),d?l.replace(b):h?(d=l,e=b.indexOf("#"),e=-1===e?"":b.substr(e),d.hash=e):l.href=b,l.href!==b&&(y=b)):(m[d?"replaceState":"pushState"](e,"",b),f());y&&(y=b);return k}return y||l.href.replace(/%27/g,"'")};k.state=function(){return F};var w=[],W=!1,U=null;k.onUrlChange=function(b){if(!W){if(c.history)x(a).on("popstate",g);x(a).on("hashchange",g);W=!0}w.push(b);return b};k.$$applicationDestroyed=function(){x(a).off("hashchange popstate",g)};k.$$checkUrlChange=
h;k.baseHref=function(){var a=kb.attr("href");return a?a.replace(/^(https?:)?\/\/[^/]*/,""):""};k.defer=function(a,b){B++;var c=n(function(){delete E[c];e(a)},b||0);E[c]=!0;return c};k.defer.cancel=function(a){return E[a]?(delete E[a],t(a),e(H),!0):!1}}function Ze(){this.$get=["$window","$log","$sniffer","$document",function(a,b,d,c){return new Ye(a,c,b,d)}]}function $e(){this.$get=function(){function a(a,c){function d(a){a!==n&&(t?t===a&&(t=a.n):t=a,g(a.n,a.p),g(a,n),n=a,n.n=null)}function g(a,b){a!==
b&&(a&&(a.p=b),b&&(b.n=a))}if(a in b)throw K("$cacheFactory")("iid",a);var f=0,h=P({},c,{id:a}),k=V(),l=c&&c.capacity||Number.MAX_VALUE,m=V(),n=null,t=null;return b[a]={put:function(a,b){if(!z(b)){if(l<Number.MAX_VALUE){var c=m[a]||(m[a]={key:a});d(c)}a in k||f++;k[a]=b;f>l&&this.remove(t.key);return b}},get:function(a){if(l<Number.MAX_VALUE){var b=m[a];if(!b)return;d(b)}return k[a]},remove:function(a){if(l<Number.MAX_VALUE){var b=m[a];if(!b)return;b===n&&(n=b.p);b===t&&(t=b.n);g(b.n,b.p);delete m[a]}a in
k&&(delete k[a],f--)},removeAll:function(){k=V();f=0;m=V();n=t=null},destroy:function(){m=h=k=null;delete b[a]},info:function(){return P({},h,{size:f})}}}var b={};a.info=function(){var a={};p(b,function(b,d){a[d]=b.info()});return a};a.get=function(a){return b[a]};return a}}function af(){this.$get=["$cacheFactory",function(a){return a("templates")}]}function nd(a,b){function d(a,b,c){var d=/^([@&<]|=(\*?))(\??)\s*([\w$]*)$/,e=V();p(a,function(a,f){a=a.trim();if(a in n)e[f]=n[a];else{var g=a.match(d);
if(!g)throw ea("iscp",b,f,a,c?"controller bindings definition":"isolate scope definition");e[f]={mode:g[1][0],collection:"*"===g[2],optional:"?"===g[3],attrName:g[4]||f};g[4]&&(n[a]=e[f])}});return e}function c(a){var b=a.charAt(0);if(!b||b!==N(b))throw ea("baddir",a);if(a!==a.trim())throw ea("baddir",a);}function e(a){var b=a.require||a.controller&&a.name;!J(b)&&G(b)&&p(b,function(a,c){var d=a.match(l);a.substring(d[0].length)||(b[c]=d[0]+c)});return b}var g={},f=/^\s*directive:\s*([\w-]+)\s+(.*)$/,
h=/(([\w-]+)(?::([^;]+))?;?)/,k=ye("ngSrc,ngSrcset,src,srcset"),l=/^(?:(\^\^?)?(\?)?(\^\^?)?)?/,m=/^(on[a-z]+|formaction)$/,n=V();this.directive=function C(b,d){jb(b,"name");Ia(b,"directive");I(b)?(c(b),jb(d,"directiveFactory"),g.hasOwnProperty(b)||(g[b]=[],a.factory(b+"Directive",["$injector","$exceptionHandler",function(a,c){var d=[];p(g[b],function(f,g){try{var h=a.invoke(f);A(h)?h={compile:la(h)}:!h.compile&&h.link&&(h.compile=la(h.link));h.priority=h.priority||0;h.index=g;h.name=h.name||b;h.require=
e(h);g=h;var k=h.restrict;if(k&&(!I(k)||!/[EACM]/.test(k)))throw ea("badrestrict",k,b);g.restrict=k||"EA";h.$$moduleName=f.$$moduleName;d.push(h)}catch(ka){c(ka)}});return d}])),g[b].push(d)):p(b,bc(C));return this};this.component=function w(a,b){function c(a){function c(b){return A(b)||J(b)?function(c,d){return a.invoke(b,this,{$element:c,$attrs:d})}:b}var e=b.template||b.templateUrl?b.template:"",f={controller:d,controllerAs:bf(b.controller)||b.controllerAs||"$ctrl",template:c(e),templateUrl:c(b.templateUrl),
transclude:b.transclude,scope:{},bindToController:b.bindings||{},restrict:"E",require:b.require};p(b,function(a,b){"$"===b.charAt(0)&&(f[b]=a)});return f}if(!I(a))return p(a,bc(Xa(this,w))),this;var d=b.controller||function(){};p(b,function(a,b){"$"===b.charAt(0)&&(c[b]=a,A(d)&&(d[b]=a))});c.$inject=["$injector"];return this.directive(a,c)};this.aHrefSanitizationWhitelist=function(a){return v(a)?(b.aHrefSanitizationWhitelist(a),this):b.aHrefSanitizationWhitelist()};this.imgSrcSanitizationWhitelist=
function(a){return v(a)?(b.imgSrcSanitizationWhitelist(a),this):b.imgSrcSanitizationWhitelist()};var t=!0;this.debugInfoEnabled=function(a){return v(a)?(t=a,this):t};var E=!1;this.preAssignBindingsEnabled=function(a){return v(a)?(E=a,this):E};var B=!1;this.strictComponentBindingsEnabled=function(a){return v(a)?(B=a,this):B};var u=10;this.onChangesTtl=function(a){return arguments.length?(u=a,this):u};var F=!0;this.commentDirectivesEnabled=function(a){return arguments.length?(F=a,this):F};var r=!0;
this.cssClassDirectivesEnabled=function(a){return arguments.length?(r=a,this):r};this.$get=["$injector","$interpolate","$exceptionHandler","$templateRequest","$parse","$controller","$rootScope","$sce","$animate","$$sanitizeUri",function(a,b,c,e,n,Ka,L,Q,R,q){function w(){try{if(!--Ha)throw Wa=void 0,ea("infchng",u);L.$apply(function(){for(var a=0,b=Wa.length;a<b;++a)try{Wa[a]()}catch(wa){c(wa)}Wa=void 0})}finally{Ha++}}function C(a,b){if(b){var c=Object.keys(b),d;var e=0;for(d=c.length;e<d;e++){var f=
c[e];this[f]=b[f]}}else this.$attr={};this.$$element=a}function y(a,b,c){Ea.innerHTML="<span "+b+">";b=Ea.firstChild.attributes;var d=b[0];b.removeNamedItem(d.name);d.value=c;a.attributes.setNamedItem(d)}function W(a,b){try{a.addClass(b)}catch(wa){}}function U(a,b,c,d,e){a instanceof x||(a=x(a));var f=ka(a,b,a,c,d,e);U.$$addScopeClass(a);var g=null;return function(b,c,d){if(!a)throw ea("multilink");jb(b,"scope");e&&e.needsNewScope&&(b=b.$parent.$new());d=d||{};var h=d.parentBoundTranscludeFn,k=d.transcludeControllers;
d=d.futureParentElement;h&&h.$$boundTransclude&&(h=h.$$boundTransclude);g||(g=(d=d&&d[0])?"foreignobject"!==za(d)&&ma.call(d).match(/SVG/)?"svg":"html":"html");d="html"!==g?x(pa(g,x("<div></div>").append(a).html())):c?Za.clone.call(a):a;if(k)for(var l in k)d.data("$"+l+"Controller",k[l].instance);U.$$addScopeInfo(d,b);c&&c(d,b);f&&f(b,d,d,h);c||(a=f=null);return d}}function ka(a,b,c,d,e,f){function g(a,c,d,e){var f,g;if(t){var k=Array(c.length);for(f=0;f<h.length;f+=3){var l=h[f];k[l]=c[l]}}else k=
c;f=0;for(g=h.length;f<g;){var n=k[h[f++]];c=h[f++];l=h[f++];if(c){if(c.scope){var m=a.$new();U.$$addScopeInfo(x(n),m)}else m=a;var w=c.transcludeOnThisElement?K(a,c.transclude,e):!c.templateOnThisElement&&e?e:!e&&b?K(a,b):null;c(l,m,n,d,w)}else l&&l(a,n.childNodes,void 0,e)}}for(var h=[],k=J(a)||a instanceof x,l,n,m,w,t,X=0;X<a.length;X++){l=new C;11===La&&pb(a,X,k);n=xc(a[X],[],l,0===X?d:void 0,e);(f=n.length?da(n,a[X],l,b,c,null,[],[],f):null)&&f.scope&&U.$$addScopeClass(l.$$element);l=f&&f.terminal||
!(m=a[X].childNodes)||!m.length?null:ka(m,f?(f.transcludeOnThisElement||!f.templateOnThisElement)&&f.transclude:b);if(f||l)h.push(X,f,l),w=!0,t=t||f;f=null}return w?g:null}function pb(a,b,c){var d=a[b],e=d.parentNode;if(d.nodeType===Sa)for(;;){var f=e?d.nextSibling:a[b+1];if(!f||f.nodeType!==Sa)break;d.nodeValue+=f.nodeValue;f.parentNode&&f.parentNode.removeChild(f);c&&f===a[b+1]&&a.splice(b+1,1)}}function K(a,b,c){function d(d,e,f,g,h){d||(d=a.$new(!1,h),d.$$transcluded=!0);return b(d,e,{parentBoundTranscludeFn:c,
transcludeControllers:f,futureParentElement:g})}var e=d.$$slots=V(),f;for(f in b.$$slots)e[f]=b.$$slots[f]?K(a,b.$$slots[f],c):null;return d}function xc(a,b,c,d,e){var f=c.$attr;switch(a.nodeType){case 1:var g=za(a);fa(b,Da(g),"E",d,e);for(var k,l,n,m,w=a.attributes,t=0,C=w&&w.length;t<C;t++){var E=!1,y=!1;k=w[t];l=k.name;n=k.value;k=Da(l);(m=Ua.test(k))&&(l=l.replace(od,"").substr(8).replace(/_(.)/g,function(a,b){return b.toUpperCase()}));(k=k.match(Va))&&na(k[1])&&(E=l,y=l.substr(0,l.length-5)+
"end",l=l.substr(0,l.length-6));k=Da(l.toLowerCase());f[k]=l;if(m||!c.hasOwnProperty(k))c[k]=n,jd(a,k)&&(c[k]=!0);Ga(a,b,n,k,m);fa(b,k,"A",d,e,E,y)}"input"===g&&"hidden"===a.getAttribute("type")&&a.setAttribute("autocomplete","off");if(!Qa)break;f=a.className;G(f)&&(f=f.animVal);if(I(f)&&""!==f)for(;a=h.exec(f);)k=Da(a[2]),fa(b,k,"C",d,e)&&(c[k]=T(a[3])),f=f.substr(a.index+a[0].length);break;case Sa:xa(b,a.nodeValue);break;case 8:Oa&&Y(a,b,c,d,e)}b.sort(ra);return b}function Y(a,b,c,d,e){try{var g=
f.exec(a.nodeValue);if(g){var h=Da(g[1]);fa(b,h,"M",d,e)&&(c[h]=T(g[2]))}}catch(wc){}}function Z(a,b,c){var d=[],e=0;if(b&&a.hasAttribute&&a.hasAttribute(b)){do{if(!a)throw ea("uterdir",b,c);1===a.nodeType&&(a.hasAttribute(b)&&e++,a.hasAttribute(c)&&e--);d.push(a);a=a.nextSibling}while(0<e)}else d.push(a);return x(d)}function ca(a,b,c){return function(d,e,f,g,h){e=Z(e[0],b,c);return a(d,e,f,g,h)}}function ba(a,b,c,d,e,f){var g;return a?U(b,c,d,e,f):function(){g||(g=U(b,c,d,e,f),b=c=f=null);return g.apply(this,
arguments)}}function da(a,b,d,e,f,g,h,k,l){function n(a,b,c,d){if(a){c&&(a=ca(a,c,d));a.require=r.require;a.directiveName=F;if(L===r||r.$$isolateScope)a=Ba(a,{isolateScope:!0});h.push(a)}if(b){c&&(b=ca(b,c,d));b.require=r.require;b.directiveName=F;if(L===r||r.$$isolateScope)b=Ba(b,{isolateScope:!0});k.push(b)}}function m(a,e,f,g,l){function m(a,b,c,d){var e;db(a)||(d=c,c=b,b=a,a=void 0);Ka&&(e=n);c||(c=Ka?w.parent():w);if(d){var f=l.$$slots[d];if(f)return f(a,b,e,c,q);if(z(f))throw ea("noslot",d,
Aa(w));}else return l(a,b,e,c,q)}var n;if(b===f){g=d;var w=d.$$element}else w=x(f),g=new C(w,d);var r=e;if(L)var u=e.$new(!0);else t&&(r=e.$parent);if(l){var B=m;B.$$boundTransclude=l;B.isSlotFilled=function(a){return!!l.$$slots[a]}}y&&(n=ja(w,g,B,y,u,e,L));if(L){U.$$addScopeInfo(w,u,!0,!(W&&(W===L||W===L.$$originalDirective)));U.$$addScopeClass(w,!0);u.$$isolateBindings=L.$$isolateBindings;var X=ta(e,g,u,u.$$isolateBindings,L);X.removeWatches&&u.$on("$destroy",X.removeWatches)}for(R in n){X=y[R];
var M=n[R];var F=X.$$bindings.bindToController;if(E){M.bindingInfo=F?ta(r,g,M.instance,F,X):{};var Q=M();Q!==M.instance&&(M.instance=Q,w.data("$"+X.name+"Controller",Q),M.bindingInfo.removeWatches&&M.bindingInfo.removeWatches(),M.bindingInfo=ta(r,g,M.instance,F,X))}else M.instance=M(),w.data("$"+X.name+"Controller",M.instance),M.bindingInfo=ta(r,g,M.instance,F,X)}p(y,function(a,b){var c=a.require;a.bindToController&&!J(c)&&G(c)&&P(n[b].instance,aa(b,c,w,n))});p(n,function(a){var b=a.instance;if(A(b.$onChanges))try{b.$onChanges(a.bindingInfo.initialChanges)}catch(yc){c(yc)}if(A(b.$onInit))try{b.$onInit()}catch(yc){c(yc)}A(b.$doCheck)&&
(r.$watch(function(){b.$doCheck()}),b.$doCheck());A(b.$onDestroy)&&r.$on("$destroy",function(){b.$onDestroy()})});var R=0;for(X=h.length;R<X;R++)M=h[R],Ca(M,M.isolateScope?u:e,w,g,M.require&&aa(M.directiveName,M.require,w,n),B);var q=e;L&&(L.template||null===L.templateUrl)&&(q=u);a&&a(q,f.childNodes,void 0,l);for(R=k.length-1;0<=R;R--)M=k[R],Ca(M,M.isolateScope?u:e,w,g,M.require&&aa(M.directiveName,M.require,w,n),B);p(n,function(a){a=a.instance;A(a.$postLink)&&a.$postLink()})}l=l||{};for(var w=-Number.MAX_VALUE,
t=l.newScopeDirective,y=l.controllerDirectives,L=l.newIsolateScopeDirective,W=l.templateDirective,u=l.nonTlbTranscludeDirective,X=!1,M=!1,Ka=l.hasElementTranscludeDirective,B=d.$$element=x(b),r,F,Q,R=e,q,wa=!1,ka=!1,v,Ta=0,Ya=a.length;Ta<Ya;Ta++){r=a[Ta];var Ma=r.$$start,vc=r.$$end;Ma&&(B=Z(b,Ma,vc));Q=void 0;if(w>r.priority)break;if(v=r.scope)r.templateUrl||(G(v)?(ha("new/isolated scope",L||t,r,B),L=r):ha("new/isolated scope",L,r,B)),t=t||r;F=r.name;if(!wa&&(r.replace&&(r.templateUrl||r.template)||
r.transclude&&!r.$$tlb)){for(v=Ta+1;wa=a[v++];)if(wa.transclude&&!wa.$$tlb||wa.replace&&(wa.templateUrl||wa.template)){ka=!0;break}wa=!0}!r.templateUrl&&r.controller&&(y=y||V(),ha("'"+F+"' controller",y[F],r,B),y[F]=r);if(v=r.transclude)if(X=!0,r.$$tlb||(ha("transclusion",u,r,B),u=r),"element"===v)Ka=!0,w=r.priority,Q=B,B=d.$$element=x(U.$$createComment(F,d[F])),b=B[0],qa(f,ya.call(Q,0),b),Q[0].$$parentNode=Q[0].parentNode,R=ba(ka,Q,e,w,g&&g.name,{nonTlbTranscludeDirective:u});else{var S=V();if(G(v)){Q=
[];var wc=V(),D=V();p(v,function(a,b){var c="?"===a.charAt(0);a=c?a.substring(1):a;wc[a]=b;S[b]=null;D[b]=c});p(B.contents(),function(a){var b=wc[Da(za(a))];b?(D[b]=!0,S[b]=S[b]||[],S[b].push(a)):Q.push(a)});p(D,function(a,b){if(!a)throw ea("reqslot",b);});for(var nb in S)S[nb]&&(S[nb]=ba(ka,S[nb],e))}else Q=x(rc(b)).contents();B.empty();R=ba(ka,Q,e,void 0,void 0,{needsNewScope:r.$$isolateScope||r.$$newScope});R.$$slots=S}if(r.template)if(M=!0,ha("template",W,r,B),W=r,v=A(r.template)?r.template(B,
d):r.template,v=Pa(v),r.replace){g=r;Q=oc.test(v)?pd(pa(r.templateNamespace,T(v))):[];b=Q[0];if(1!==Q.length||1!==b.nodeType)throw ea("tplrt",F,"");qa(f,B,b);Ya={$attr:{}};v=xc(b,[],Ya);var H=a.splice(Ta+1,a.length-(Ta+1));(L||t)&&ia(v,L,t);a=a.concat(v).concat(H);la(d,Ya);Ya=a.length}else B.html(v);if(r.templateUrl)M=!0,ha("template",W,r,B),W=r,r.replace&&(g=r),m=oa(a.splice(Ta,a.length-Ta),B,d,f,X&&R,h,k,{controllerDirectives:y,newScopeDirective:t!==r&&t,newIsolateScopeDirective:L,templateDirective:W,
nonTlbTranscludeDirective:u}),Ya=a.length;else if(r.compile)try{q=r.compile(B,d,R);var pb=r.$$originalDirective||r;A(q)?n(null,Xa(pb,q),Ma,vc):q&&n(Xa(pb,q.pre),Xa(pb,q.post),Ma,vc)}catch(ef){c(ef,Aa(B))}r.terminal&&(m.terminal=!0,w=Math.max(w,r.priority))}m.scope=t&&!0===t.scope;m.transcludeOnThisElement=X;m.templateOnThisElement=M;m.transclude=R;l.hasElementTranscludeDirective=Ka;return m}function aa(a,b,c,d){if(I(b)){var e=b.match(l);b=b.substring(e[0].length);var f=e[1]||e[3];e="?"===e[2];if("^^"===
f)c=c.parent();else var g=(g=d&&d[b])&&g.instance;if(!g){var h="$"+b+"Controller";g=f?c.inheritedData(h):c.data(h)}if(!g&&!e)throw ea("ctreq",b,a);}else if(J(b))for(g=[],f=0,e=b.length;f<e;f++)g[f]=aa(a,b[f],c,d);else G(b)&&(g={},p(b,function(b,e){g[e]=aa(a,b,c,d)}));return g||null}function ja(a,b,c,d,e,f,g){var h=V(),k;for(k in d){var l=d[k],n={$scope:l===g||l.$$isolateScope?e:f,$element:a,$attrs:b,$transclude:c},w=l.controller;"@"===w&&(w=b[l.name]);n=Ka(w,n,!0,l.controllerAs);h[l.name]=n;a.data("$"+
l.name+"Controller",n.instance)}return h}function ia(a,b,c){for(var d=0,e=a.length;d<e;d++)a[d]=ec(a[d],{$$isolateScope:b,$$newScope:c})}function fa(b,c,e,f,h,k,l){if(c===h)return null;var n=null;if(g.hasOwnProperty(c)){h=a.get(c+"Directive");for(var w=0,m=h.length;w<m;w++)if(c=h[w],(z(f)||f>c.priority)&&-1!==c.restrict.indexOf(e)){k&&(c=ec(c,{$$start:k,$$end:l}));if(!c.$$bindings){var t=n=c,C=c.name,r={isolateScope:null,bindToController:null};G(t.scope)&&(!0===t.bindToController?(r.bindToController=
d(t.scope,C,!0),r.isolateScope={}):r.isolateScope=d(t.scope,C,!1));G(t.bindToController)&&(r.bindToController=d(t.bindToController,C,!0));if(r.bindToController&&!t.controller)throw ea("noctrl",C);n=n.$$bindings=r;G(n.isolateScope)&&(c.$$isolateBindings=n.isolateScope)}b.push(c);n=c}}return n}function na(b){if(g.hasOwnProperty(b))for(var c=a.get(b+"Directive"),d=0,e=c.length;d<e;d++)if(b=c[d],b.multiElement)return!0;return!1}function la(a,b){var c=b.$attr,d=a.$attr;p(a,function(d,e){"$"!==e.charAt(0)&&
(b[e]&&b[e]!==d&&(d=d.length?d+(("style"===e?";":" ")+b[e]):b[e]),a.$set(e,d,!0,c[e]))});p(b,function(b,e){a.hasOwnProperty(e)||"$"===e.charAt(0)||(a[e]=b,"class"!==e&&"style"!==e&&(d[e]=c[e]))})}function oa(a,b,d,f,g,h,k,l){var n=[],w,m,t=b[0],C=a.shift(),r=ec(C,{templateUrl:null,transclude:null,replace:null,$$originalDirective:C}),L=A(C.templateUrl)?C.templateUrl(b,d):C.templateUrl,y=C.templateNamespace;b.empty();e(L).then(function(c){c=Pa(c);if(C.replace){c=oc.test(c)?pd(pa(y,T(c))):[];var e=c[0];
if(1!==c.length||1!==e.nodeType)throw ea("tplrt",C.name,L);c={$attr:{}};qa(f,b,e);var E=xc(e,[],c);G(C.scope)&&ia(E,!0);a=E.concat(a);la(d,c)}else e=t,b.html(c);a.unshift(r);w=da(a,e,d,g,b,C,h,k,l);p(f,function(a,c){a===e&&(f[c]=b[0])});for(m=ka(b[0].childNodes,g);n.length;){c=n.shift();var u=n.shift();var B=n.shift(),U=n.shift();E=b[0];if(!c.$$destroyed){if(u!==t){var F=u.className;l.hasElementTranscludeDirective&&C.replace||(E=rc(e));qa(B,x(u),E);W(x(E),F)}u=w.transcludeOnThisElement?K(c,w.transclude,
U):U;w(m,c,E,f,u)}}n=null}).catch(function(a){gc(a)&&c(a)});return function(a,b,c,d,e){a=e;b.$$destroyed||(n?n.push(b,c,d,a):(w.transcludeOnThisElement&&(a=K(b,w.transclude,e)),w(m,b,c,d,a)))}}function ra(a,b){var c=b.priority-a.priority;return 0!==c?c:a.name!==b.name?a.name<b.name?-1:1:a.index-b.index}function ha(a,b,c,d){function e(a){return a?" (module: "+a+")":""}if(b)throw ea("multidir",b.name,e(b.$$moduleName),c.name,e(c.$$moduleName),a,Aa(d));}function xa(a,c){var d=b(c,!0);d&&a.push({priority:0,
compile:function(a){a=a.parent();var b=!!a.length;b&&U.$$addBindingClass(a);return function(a,c){var e=c.parent();b||U.$$addBindingClass(e);U.$$addBindingInfo(e,d.expressions);a.$watch(d,function(a){c[0].nodeValue=a})}}})}function pa(a,b){a=N(a||"html");switch(a){case "svg":case "math":var c=D.document.createElement("div");c.innerHTML="<"+a+">"+b+"</"+a+">";return c.childNodes[0].childNodes;default:return b}}function Fa(a,b){if("srcdoc"===b)return Q.HTML;a=za(a);if("src"===b||"ngSrc"===b){if(-1===
["img","video","audio","source","track"].indexOf(a))return Q.RESOURCE_URL}else if("xlinkHref"===b||"form"===a&&"action"===b||"link"===a&&"href"===b)return Q.RESOURCE_URL}function Ga(a,c,d,e,f){var g=Fa(a,e),h=k[e]||f,l=b(d,!f,g,h);if(l){if("multiple"===e&&"select"===za(a))throw ea("selmulti",Aa(a));if(m.test(e))throw ea("nodomevents");c.push({priority:100,compile:function(){return{pre:function(a,c,f){c=f.$$observers||(f.$$observers=V());var k=f[e];k!==d&&(l=k&&b(k,!0,g,h),d=k);l&&(f[e]=l(a),(c[e]||
(c[e]=[])).$$inter=!0,(f.$$observers&&f.$$observers[e].$$scope||a).$watch(l,function(a,b){"class"===e&&a!==b?f.$updateClass(a,b):f.$set(e,a)}))}}}})}}function qa(a,b,c){var d=b[0],e=b.length,f=d.parentNode,g;if(a){var h=0;for(g=a.length;h<g;h++)if(a[h]===d){a[h++]=c;g=h+e-1;for(var k=a.length;h<k;h++,g++)g<k?a[h]=a[g]:delete a[h];a.length-=e-1;a.context===d&&(a.context=c);break}}f&&f.replaceChild(c,d);a=D.document.createDocumentFragment();for(h=0;h<e;h++)a.appendChild(b[h]);x.hasData(d)&&(x.data(c,
x.data(d)),x(d).off("$destroy"));x.cleanData(a.querySelectorAll("*"));for(h=1;h<e;h++)delete b[h];b[0]=c;b.length=1}function Ba(a,b){return P(function(){return a.apply(null,arguments)},a,b)}function Ca(a,b,d,e,f,g){try{a(b,d,e,f,g)}catch(cf){c(cf,Aa(d))}}function sa(a,b){if(B)throw ea("missingattr",a,b);}function ta(a,c,d,e,f){function g(b,c,e){A(d.$onChanges)&&!hc(c,e)&&(Wa||(a.$$postDigest(w),Wa=[]),m||(m={},Wa.push(h)),m[b]&&(e=m[b].previousValue),m[b]=new Mb(e,c))}function h(){d.$onChanges(m);
m=void 0}var k=[],l={},m;p(e,function(e,h){var w=e.attrName,m=e.optional;switch(e.mode){case "@":m||ua.call(c,w)||(sa(w,f.name),d[h]=c[w]=void 0);e=c.$observe(w,function(a){if(I(a)||Ra(a))g(h,a,d[h]),d[h]=a});c.$$observers[w].$$scope=a;var t=c[w];I(t)?d[h]=b(t)(a):Ra(t)&&(d[h]=t);l[h]=new Mb(zc,d[h]);k.push(e);break;case "=":if(!ua.call(c,w)){if(m)break;sa(w,f.name);c[w]=void 0}if(m&&!c[w])break;var C=n(c[w]);var r=C.literal?va:hc;var E=C.assign||function(){t=d[h]=C(a);throw ea("nonassign",c[w],w,
f.name);};t=d[h]=C(a);m=function(b){r(b,d[h])||(r(b,t)?E(a,b=d[h]):d[h]=b);return t=b};m.$stateful=!0;e=e.collection?a.$watchCollection(c[w],m):a.$watch(n(c[w],m),null,C.literal);k.push(e);break;case "<":if(!ua.call(c,w)){if(m)break;sa(w,f.name);c[w]=void 0}if(m&&!c[w])break;C=n(c[w]);var L=C.literal,y=d[h]=C(a);l[h]=new Mb(zc,d[h]);e=a.$watch(C,function(a,b){if(b===a){if(b===y||L&&va(b,y))return;b=y}g(h,a,b);d[h]=a},L);k.push(e);break;case "&":m||ua.call(c,w)||sa(w,f.name),C=c.hasOwnProperty(w)?
n(c[w]):H,C===H&&m||(d[h]=function(b){return C(a,b)})}});return{initialChanges:l,removeWatches:k.length&&function(){for(var a=0,b=k.length;a<b;++a)k[a]()}}}var Na=/^\w/,Ea=D.document.createElement("div"),Oa=F,Qa=r,Ha=u,Wa;C.prototype={$normalize:Da,$addClass:function(a){a&&0<a.length&&R.addClass(this.$$element,a)},$removeClass:function(a){a&&0<a.length&&R.removeClass(this.$$element,a)},$updateClass:function(a,b){var c=qd(a,b);c&&c.length&&R.addClass(this.$$element,c);(a=qd(b,a))&&a.length&&R.removeClass(this.$$element,
a)},$set:function(a,b,d,e){var f=jd(this.$$element[0],a),g=rd[a],h=a;f?(this.$$element.prop(a,b),e=f):g&&(this[g]=b,h=g);this[a]=b;e?this.$attr[a]=e:(e=this.$attr[a])||(this.$attr[a]=e=cd(a,"-"));f=za(this.$$element);if("a"===f&&("href"===a||"xlinkHref"===a)||"img"===f&&"src"===a)this[a]=b=null==b?b:q(b,"src"===a);else if("img"===f&&"srcset"===a&&v(b)){f="";g=T(b);var k=/(\s+\d+x\s*,|\s+\d+w\s*,|\s+,|,\s+)/;k=/\s/.test(g)?k:/(,)/;g=g.split(k);k=Math.floor(g.length/2);for(var l=0;l<k;l++){var w=2*
l;f+=q(T(g[w]),!0);f+=" "+T(g[w+1])}g=T(g[2*l]).split(/\s/);f+=q(T(g[0]),!0);2===g.length&&(f+=" "+T(g[1]));this[a]=b=f}!1!==d&&(null==b?this.$$element.removeAttr(e):Na.test(e)?this.$$element.attr(e,b):y(this.$$element[0],e,b));(a=this.$$observers)&&p(a[h],function(a){try{a(b)}catch(df){c(df)}})},$observe:function(a,b){var c=this,d=c.$$observers||(c.$$observers=V()),e=d[a]||(d[a]=[]);e.push(b);L.$evalAsync(function(){e.$$inter||!c.hasOwnProperty(a)||z(c[a])||b(c[a])});return function(){eb(e,b)}}};
var Ia=b.startSymbol(),Ja=b.endSymbol(),Pa="{{"===Ia&&"}}"===Ja?yb:function(a){return a.replace(/\{\{/g,Ia).replace(/}}/g,Ja)},Ua=/^ngAttr[A-Z]/,Va=/^(.+)Start$/;U.$$addBindingInfo=t?function(a,b){var c=a.data("$binding")||[];J(b)?c=c.concat(b):c.push(b);a.data("$binding",c)}:H;U.$$addBindingClass=t?function(a){W(a,"ng-binding")}:H;U.$$addScopeInfo=t?function(a,b,c,d){a.data(c?d?"$isolateScopeNoTemplate":"$isolateScope":"$scope",b)}:H;U.$$addScopeClass=t?function(a,b){W(a,b?"ng-isolate-scope":"ng-scope")}:
H;U.$$createComment=function(a,b){var c="";t&&(c=" "+(a||"")+": ",b&&(c+=b+" "));return D.document.createComment(c)};return U}]}function Mb(a,b){this.previousValue=a;this.currentValue=b}function Da(a){return a.replace(od,"").replace(ff,function(a,d,c){return c?d.toUpperCase():d})}function qd(a,b){var d="";a=a.split(/\s+/);b=b.split(/\s+/);var c=0;a:for(;c<a.length;c++){for(var e=a[c],g=0;g<b.length;g++)if(e===b[g])continue a;d+=(0<d.length?" ":"")+e}return d}function pd(a){a=x(a);var b=a.length;if(1>=
b)return a;for(;b--;){var d=a[b];(8===d.nodeType||d.nodeType===Sa&&""===d.nodeValue.trim())&&gf.call(a,b,1)}return a}function bf(a,b){if(b&&I(b))return b;if(I(a)&&(a=sd.exec(a)))return a[3]}function hf(){var a={},b=!1;this.has=function(b){return a.hasOwnProperty(b)};this.register=function(b,c){Ia(b,"controller");G(b)?P(a,b):a[b]=c};this.allowGlobals=function(){b=!0};this.$get=["$injector","$window",function(d,c){function e(a,b,c,d){if(!a||!G(a.$scope))throw K("$controller")("noscp",d,b);a.$scope[b]=
c}return function(g,f,h,k){var l;h=!0===h;k&&I(k)&&(l=k);if(I(g)){k=g.match(sd);if(!k)throw td("ctrlfmt",g);var m=k[1];l=l||k[3];g=a.hasOwnProperty(m)?a[m]:dd(f.$scope,m,!0)||(b?dd(c,m,!0):void 0);if(!g)throw td("ctrlreg",m);zb(g,m,!0)}if(h){h=(J(g)?g[g.length-1]:g).prototype;var n=Object.create(h||null);l&&e(f,l,n,m||g.name);return P(function(){var a=d.invoke(g,n,f,m);a!==n&&(G(a)||A(a))&&(n=a,l&&e(f,l,n,m||g.name));return n},{instance:n,identifier:l})}n=d.instantiate(g,f,m);l&&e(f,l,n,m||g.name);
return n}}]}function jf(){this.$get=["$window",function(a){return x(a.document)}]}function kf(){this.$get=["$document","$rootScope",function(a,b){function d(){e=c.hidden}var c=a[0],e=c&&c.hidden;a.on("visibilitychange",d);b.$on("$destroy",function(){a.off("visibilitychange",d)});return function(){return e}}]}function lf(){this.$get=["$log",function(a){return function(b,d){a.error.apply(a,arguments)}}]}function Ac(a){return G(a)?ia(a)?a.toISOString():gb(a):a}function mf(){this.$get=function(){return function(a){if(!a)return"";
var b=[];Wc(a,function(a,c){null===a||z(a)||A(a)||(J(a)?p(a,function(a){b.push(na(c)+"="+na(Ac(a)))}):b.push(na(c)+"="+na(Ac(a))))});return b.join("&")}}}function nf(){this.$get=function(){return function(a){function b(a,e,g){null===a||z(a)||(J(a)?p(a,function(a,c){b(a,e+"["+(G(a)?c:"")+"]")}):G(a)&&!ia(a)?Wc(a,function(a,c){b(a,e+(g?"":"[")+c+(g?"":"]"))}):d.push(na(e)+"="+na(Ac(a))))}if(!a)return"";var d=[];b(a,"",!0);return d.join("&")}}}function Bc(a,b){if(I(a)){var d=a.replace(of,"").trim();
if(d){b=(b=b("Content-Type"))&&0===b.indexOf(ud);var c;(c=b)||(c=(c=d.match(pf))&&qf[c[0]].test(d));if(c)try{a=Zc(d)}catch(e){if(!b)return a;throw Nb("baddata",a,e);}}}return a}function vd(a){var b=V(),d;I(a)?p(a.split("\n"),function(a){d=a.indexOf(":");var c=N(T(a.substr(0,d)));a=T(a.substr(d+1));c&&(b[c]=b[c]?b[c]+", "+a:a)}):G(a)&&p(a,function(a,d){d=N(d);a=T(a);d&&(b[d]=b[d]?b[d]+", "+a:a)});return b}function wd(a){var b;return function(d){b||(b=vd(a));return d?(d=b[N(d)],void 0===d&&(d=null),
d):b}}function xd(a,b,d,c){if(A(c))return c(a,b,d);p(c,function(c){a=c(a,b,d)});return a}function rf(){var a=this.defaults={transformResponse:[Bc],transformRequest:[function(a){return G(a)&&"[object File]"!==ma.call(a)&&"[object Blob]"!==ma.call(a)&&"[object FormData]"!==ma.call(a)?gb(a):a}],headers:{common:{Accept:"application/json, text/plain, */*"},post:ra(Cc),put:ra(Cc),patch:ra(Cc)},xsrfCookieName:"XSRF-TOKEN",xsrfHeaderName:"X-XSRF-TOKEN",paramSerializer:"$httpParamSerializer",jsonpCallbackParam:"callback"},
b=!1;this.useApplyAsync=function(a){return v(a)?(b=!!a,this):b};var d=this.interceptors=[],c=this.xsrfWhitelistedOrigins=[];this.$get=["$browser","$httpBackend","$$cookieReader","$cacheFactory","$rootScope","$q","$injector","$sce",function(e,g,f,h,k,l,m,n){function t(b){function c(a,b){for(var c=0,d=b.length;c<d;){var e=b[c++],f=b[c++];a=a.then(e,f)}b.length=0;return a}function d(a,b){var c,d={};p(a,function(a,e){A(a)?(c=a(b),null!=c&&(d[e]=c)):d[e]=a});return d}function f(a){var b=P({},a);b.data=
xd(a.data,a.headers,a.status,g.transformResponse);a=a.status;return 200<=a&&300>a?b:l.reject(b)}if(!G(b))throw K("$http")("badreq",b);if(!I(n.valueOf(b.url)))throw K("$http")("badreq",b.url);var g=P({method:"get",transformRequest:a.transformRequest,transformResponse:a.transformResponse,paramSerializer:a.paramSerializer,jsonpCallbackParam:a.jsonpCallbackParam},b);g.headers=function(b){var c=a.headers,e=P({},b.headers),f,g;c=P({},c.common,c[N(b.method)]);a:for(f in c){var h=N(f);for(g in e)if(N(g)===
h)continue a;e[f]=c[f]}return d(e,ra(b))}(b);g.method=Ob(g.method);g.paramSerializer=I(g.paramSerializer)?m.get(g.paramSerializer):g.paramSerializer;e.$$incOutstandingRequestCount();var h=[],k=[];b=l.resolve(g);p(r,function(a){(a.request||a.requestError)&&h.unshift(a.request,a.requestError);(a.response||a.responseError)&&k.push(a.response,a.responseError)});b=c(b,h);b=b.then(function(b){var c=b.headers,d=xd(b.data,wd(c),void 0,b.transformRequest);z(d)&&p(c,function(a,b){"content-type"===N(b)&&delete c[b]});
z(b.withCredentials)&&!z(a.withCredentials)&&(b.withCredentials=a.withCredentials);return E(b,d).then(f,f)});b=c(b,k);return b=b.finally(function(){e.$$completeOutstandingRequest(H)})}function E(c,d){function e(a){if(a){var c={};p(a,function(a,d){c[d]=function(c){function d(){a(c)}b?k.$applyAsync(d):k.$$phase?d():k.$apply(d)}});return c}}function h(a,c,d,e,f){function g(){m(c,a,d,e,f)}q&&(200<=a&&300>a?q.put(S,[a,c,vd(d),e,f]):q.remove(S));b?k.$applyAsync(g):(g(),k.$$phase||k.$apply())}function m(a,
b,d,e,f){b=-1<=b?b:0;(200<=b&&300>b?y.resolve:y.reject)({data:a,status:b,headers:wd(d),config:c,statusText:e,xhrStatus:f})}function r(a){m(a.data,a.status,ra(a.headers()),a.statusText,a.xhrStatus)}function E(){var a=t.pendingRequests.indexOf(c);-1!==a&&t.pendingRequests.splice(a,1)}var y=l.defer(),kb=y.promise,q,ka=c.headers,Ma="jsonp"===N(c.method),S=c.url;Ma?S=n.getTrustedResourceUrl(S):I(S)||(S=n.valueOf(S));S=B(S,c.paramSerializer(c.params));Ma&&(S=u(S,c.jsonpCallbackParam));t.pendingRequests.push(c);
kb.then(E,E);!c.cache&&!a.cache||!1===c.cache||"GET"!==c.method&&"JSONP"!==c.method||(q=G(c.cache)?c.cache:G(a.cache)?a.cache:F);if(q){var x=q.get(S);v(x)?x&&A(x.then)?x.then(r,r):J(x)?m(x[1],x[0],ra(x[2]),x[3],x[4]):m(x,200,{},"OK","complete"):q.put(S,kb)}z(x)&&((x=O(c.url)?f()[c.xsrfCookieName||a.xsrfCookieName]:void 0)&&(ka[c.xsrfHeaderName||a.xsrfHeaderName]=x),g(c.method,S,d,h,ka,c.timeout,c.withCredentials,c.responseType,e(c.eventHandlers),e(c.uploadEventHandlers)));return kb}function B(a,b){0<
b.length&&(a+=(-1===a.indexOf("?")?"?":"&")+b);return a}function u(a,b){var c=a.split("?");if(2<c.length)throw Nb("badjsonp",a);c=kc(c[1]);p(c,function(c,d){if("JSON_CALLBACK"===c)throw Nb("badjsonp",a);if(d===b)throw Nb("badjsonp",b,a);});return a+=(-1===a.indexOf("?")?"?":"&")+b+"=JSON_CALLBACK"}var F=h("$http");a.paramSerializer=I(a.paramSerializer)?m.get(a.paramSerializer):a.paramSerializer;var r=[];p(d,function(a){r.unshift(I(a)?m.get(a):m.invoke(a))});var O=sf(c);t.pendingRequests=[];(function(a){p(arguments,
function(a){t[a]=function(b,c){return t(P({},c||{},{method:a,url:b}))}})})("get","delete","head","jsonp");(function(a){p(arguments,function(a){t[a]=function(b,c,d){return t(P({},d||{},{method:a,url:b,data:c}))}})})("post","put","patch");t.defaults=a;return t}]}function tf(){this.$get=function(){return function(){return new D.XMLHttpRequest}}}function uf(){this.$get=["$browser","$jsonpCallbacks","$document","$xhrFactory",function(a,b,d,c){return vf(a,c,a.defer,b,d[0])}]}function vf(a,b,d,c,e){function g(a,
b,d){a=a.replace("JSON_CALLBACK",b);var f=e.createElement("script"),g=null;f.type="text/javascript";f.src=a;f.async=!0;g=function(a){f.removeEventListener("load",g);f.removeEventListener("error",g);e.body.removeChild(f);f=null;var h=-1,k="unknown";a&&("load"!==a.type||c.wasCalled(b)||(a={type:"error"}),k=a.type,h="error"===a.type?404:200);d&&d(h,k)};f.addEventListener("load",g);f.addEventListener("error",g);e.body.appendChild(f);return g}return function(e,h,k,l,m,n,t,E,B,u){function f(a){C="timeout"===
a;q&&q();y&&y.abort()}function r(a,b,c,e,f,g){v(w)&&d.cancel(w);q=y=null;a(b,c,e,f,g)}h=h||a.url();if("jsonp"===N(e))var O=c.createCallback(h),q=g(h,O,function(a,b){var d=200===a&&c.getResponse(O);r(l,a,d,"",b,"complete");c.removeCallback(O)});else{var y=b(e,h),C=!1;y.open(e,h,!0);p(m,function(a,b){v(a)&&y.setRequestHeader(b,a)});y.onload=function(){var a=y.statusText||"",b="response"in y?y.response:y.responseText,c=1223===y.status?204:y.status;0===c&&(c=b?200:"file"===ha(h).protocol?404:0);r(l,c,
b,y.getAllResponseHeaders(),a,"complete")};y.onerror=function(){r(l,-1,null,null,"","error")};y.ontimeout=function(){r(l,-1,null,null,"","timeout")};y.onabort=function(){r(l,-1,null,null,"",C?"timeout":"abort")};p(B,function(a,b){y.addEventListener(b,a)});p(u,function(a,b){y.upload.addEventListener(b,a)});t&&(y.withCredentials=!0);if(E)try{y.responseType=E}catch(W){if("json"!==E)throw W;}y.send(z(k)?null:k)}if(0<n)var w=d(function(){f("timeout")},n);else n&&A(n.then)&&n.then(function(){f(v(n.$$timeoutId)?
"timeout":"abort")})}}function wf(){var a="{{",b="}}";this.startSymbol=function(b){return b?(a=b,this):a};this.endSymbol=function(a){return a?(b=a,this):b};this.$get=["$parse","$exceptionHandler","$sce",function(d,c,e){function g(a){return"\\\\\\"+a}function f(c){return c.replace(n,a).replace(t,b)}function h(a,b,c,d){var e=a.$watch(function(a){e();return d(a)},b,c);return e}function k(g,k,n,t){function r(a){try{var b=a;a=n?e.getTrusted(n,b):e.valueOf(b);return t&&!v(a)?a:mc(a)}catch(ka){c(Na.interr(g,
ka))}}if(!g.length||-1===g.indexOf(a)){if(!k){k=f(g);var E=la(k);E.exp=g;E.expressions=[];E.$$watchDelegate=h}return E}t=!!t;var u,y,C=0,w=[],B=[];E=g.length;for(var F=[],p=[];C<E;)if(-1!==(u=g.indexOf(a,C))&&-1!==(y=g.indexOf(b,u+l)))C!==u&&F.push(f(g.substring(C,u))),C=g.substring(u+l,y),w.push(C),B.push(d(C,r)),C=y+m,p.push(F.length),F.push("");else{C!==E&&F.push(f(g.substring(C)));break}n&&1<F.length&&Na.throwNoconcat(g);if(!k||w.length){var L=function(a){for(var b=0,c=w.length;b<c;b++){if(t&&
z(a[b]))return;F[p[b]]=a[b]}return F.join("")};return P(function(a){var b=0,d=w.length,e=Array(d);try{for(;b<d;b++)e[b]=B[b](a);return L(e)}catch(S){c(Na.interr(g,S))}},{exp:g,expressions:w,$$watchDelegate:function(a,b){var c;return a.$watchGroup(B,function(d,e){var f=L(d);b.call(this,f,d!==e?c:f,a);c=f})}})}}var l=a.length,m=b.length,n=new RegExp(a.replace(/./g,g),"g"),t=new RegExp(b.replace(/./g,g),"g");k.startSymbol=function(){return a};k.endSymbol=function(){return b};return k}]}function xf(){this.$get=
["$rootScope","$window","$q","$$q","$browser",function(a,b,d,c,e){function g(g,k,l,m){function h(){t?g.apply(null,E):g(F)}var t=4<arguments.length,E=t?ya.call(arguments,4):[],B=b.setInterval,u=b.clearInterval,F=0,r=v(m)&&!m,O=(r?c:d).defer(),p=O.promise;l=v(l)?l:0;p.$$intervalId=B(function(){r?e.defer(h):a.$evalAsync(h);O.notify(F++);0<l&&F>=l&&(O.resolve(F),u(p.$$intervalId),delete f[p.$$intervalId]);r||a.$apply()},k);f[p.$$intervalId]=O;return p}var f={};g.cancel=function(a){return a&&a.$$intervalId in
f?(f[a.$$intervalId].promise.$$state.pur=!0,f[a.$$intervalId].reject("canceled"),b.clearInterval(a.$$intervalId),delete f[a.$$intervalId],!0):!1};return g}]}function Dc(a){a=a.split("/");for(var b=a.length;b--;)a[b]=hb(a[b].replace(/%2F/g,"/"));return a.join("/")}function yd(a,b){a=ha(a);b.$$protocol=a.protocol;b.$$host=a.hostname;b.$$port=parseInt(a.port,10)||yf[a.protocol]||null}function zd(a,b,d){if(zf.test(a))throw qb("badpath",a);var c="/"!==a.charAt(0);c&&(a="/"+a);a=ha(a);c=(c&&"/"===a.pathname.charAt(0)?
a.pathname.substring(1):a.pathname).split("/");for(var e=c.length;e--;)c[e]=decodeURIComponent(c[e]),d&&(c[e]=c[e].replace(/\//g,"%2F"));d=c.join("/");b.$$path=d;b.$$search=kc(a.search);b.$$hash=decodeURIComponent(a.hash);b.$$path&&"/"!==b.$$path.charAt(0)&&(b.$$path="/"+b.$$path)}function Ec(a,b){return a.slice(0,b.length)===b}function xa(a,b){if(Ec(b,a))return b.substr(a.length)}function ta(a){var b=a.indexOf("#");return-1===b?a:a.substr(0,b)}function rb(a){return a.replace(/(#.+)|#$/,"$1")}function Fc(a,
b,d){this.$$html5=!0;d=d||"";yd(a,this);this.$$parse=function(a){var c=xa(b,a);if(!I(c))throw qb("ipthprfx",a,b);zd(c,this,!0);this.$$path||(this.$$path="/");this.$$compose()};this.$$compose=function(){var a=lc(this.$$search),d=this.$$hash?"#"+hb(this.$$hash):"";this.$$url=Dc(this.$$path)+(a?"?"+a:"")+d;this.$$absUrl=b+this.$$url.substr(1);this.$$urlUpdatedByLocation=!0};this.$$parseLinkUrl=function(c,e){if(e&&"#"===e[0])return this.hash(e.slice(1)),!0;if(v(e=xa(a,c))){c=e;var g=d&&v(e=xa(d,e))?b+
(xa("/",e)||e):a+c}else v(e=xa(b,c))?g=b+e:b===c+"/"&&(g=b);g&&this.$$parse(g);return!!g}}function Gc(a,b,d){yd(a,this);this.$$parse=function(c){var e=xa(a,c)||xa(b,c);if(z(e)||"#"!==e.charAt(0))if(this.$$html5)var g=e;else g="",z(e)&&(a=c,this.replace());else g=xa(d,e),z(g)&&(g=e);zd(g,this,!1);c=this.$$path;e=a;var f=/^\/[A-Z]:(\/.*)/;Ec(g,e)&&(g=g.replace(e,""));f.exec(g)||(c=(g=f.exec(c))?g[1]:c);this.$$path=c;this.$$compose()};this.$$compose=function(){var b=lc(this.$$search),e=this.$$hash?"#"+
hb(this.$$hash):"";this.$$url=Dc(this.$$path)+(b?"?"+b:"")+e;this.$$absUrl=a+(this.$$url?d+this.$$url:"");this.$$urlUpdatedByLocation=!0};this.$$parseLinkUrl=function(b,d){return ta(a)===ta(b)?(this.$$parse(b),!0):!1}}function Ad(a,b,d){this.$$html5=!0;Gc.apply(this,arguments);this.$$parseLinkUrl=function(c,e){if(e&&"#"===e[0])return this.hash(e.slice(1)),!0;var g,f;a===ta(c)?g=c:(f=xa(b,c))?g=a+d+f:b===c+"/"&&(g=b);g&&this.$$parse(g);return!!g};this.$$compose=function(){var b=lc(this.$$search),e=
this.$$hash?"#"+hb(this.$$hash):"";this.$$url=Dc(this.$$path)+(b?"?"+b:"")+e;this.$$absUrl=a+d+this.$$url;this.$$urlUpdatedByLocation=!0}}function Pb(a){return function(){return this[a]}}function Bd(a,b){return function(d){if(z(d))return this[a];this[a]=b(d);this.$$compose();return this}}function Af(){var a="!",b={enabled:!1,requireBase:!0,rewriteLinks:!0},d=function(a,b,d){return a!==b};this.hashPrefix=function(b){return v(b)?(a=b,this):a};this.html5Mode=function(a){if(Ra(a))return b.enabled=a,this;
if(G(a)){Ra(a.enabled)&&(b.enabled=a.enabled);Ra(a.requireBase)&&(b.requireBase=a.requireBase);if(Ra(a.rewriteLinks)||I(a.rewriteLinks))b.rewriteLinks=a.rewriteLinks;return this}return b};this.compareUrls=function(a){return v(a)?(d=a,this):d};this.$get=["$rootScope","$browser","$sniffer","$rootElement","$window",function(c,e,g,f,h){function k(a,b,c){var d=u.url(),f=u.$$state;try{e.url(a,b,c),u.$$state=e.state()}catch(W){throw u.url(d),u.$$state=f,W;}}function l(a,b){c.$broadcast("$locationChangeSuccess",
u.absUrl(),a,u.$$state,b)}var m=e.baseHref(),n=e.url();if(b.enabled){if(!m&&b.requireBase)throw qb("nobase");var t=n.substring(0,n.indexOf("/",n.indexOf("//")+2))+(m||"/");var E=g.history?Fc:Ad}else t=ta(n),E=Gc;var B=t.substr(0,ta(t).lastIndexOf("/")+1);var u=new E(t,B,"#"+a);u.$$parseLinkUrl(n,n);u.$$state=e.state();var F=/^\s*(javascript|mailto):/i;f.on("click",function(a){var d=b.rewriteLinks;if(d&&!a.ctrlKey&&!a.metaKey&&!a.shiftKey&&2!==a.which&&2!==a.button){for(var g=x(a.target);"a"!==za(g[0]);)if(g[0]===
f[0]||!(g=g.parent())[0])return;if(!I(d)||!z(g.attr(d))){d=g.prop("href");var k=g.attr("href")||g.attr("xlink:href");G(d)&&"[object SVGAnimatedString]"===d.toString()&&(d=ha(d.animVal).href);F.test(d)||!d||g.attr("target")||a.isDefaultPrevented()||!u.$$parseLinkUrl(d,k)||(a.preventDefault(),u.absUrl()!==e.url()&&(c.$apply(),h.angular["ff-684208-preventDefault"]=!0))}}});rb(u.absUrl())!==rb(n)&&e.url(u.absUrl(),!0);var r=!0;e.onUrlChange(function(a,b){Ec(a,B)?(c.$evalAsync(function(){var d=u.absUrl(),
e=u.$$state;a=rb(a);u.$$parse(a);u.$$state=b;var f=c.$broadcast("$locationChangeStart",a,d,b,e).defaultPrevented;u.absUrl()===a&&(f?(u.$$parse(d),u.$$state=e,k(d,!1,e)):(r=!1,l(d,e)))}),c.$$phase||c.$digest()):h.location.href=a});c.$watch(function(){if(r||u.$$urlUpdatedByLocation){u.$$urlUpdatedByLocation=!1;var b=rb(e.url()),f=rb(u.absUrl()),h=e.state(),m=u.$$replace,w=d(b,f,function(){return new E(t,B,"#"+a)}),n=u.$$html5&&g.history&&h!==u.$$state,F=b!==f||n;if(r||F)r=!1,c.$evalAsync(function(){var a=
u.absUrl(),d=c.$broadcast("$locationChangeStart",a,b,u.$$state,h).defaultPrevented;u.absUrl()===a&&(d?(u.$$parse(b),u.$$state=h):((r||F&&w)&&k(a,m,h===u.$$state?null:u.$$state),l(b,h)))})}u.$$replace=!1});return u}]}function Bf(){var a=!0,b=this;this.debugEnabled=function(b){return v(b)?(a=b,this):a};this.$get=["$window",function(d){function c(a){gc(a)&&(a.stack&&g?a=a.message&&-1===a.stack.indexOf(a.message)?"Error: "+a.message+"\n"+a.stack:a.stack:a.sourceURL&&(a=a.message+"\n"+a.sourceURL+":"+
a.line));return a}function e(a){var b=d.console||{},e=b[a]||b.log||H;return function(){var a=[];p(arguments,function(b){a.push(c(b))});return Function.prototype.apply.call(e,b,a)}}var g=La||/\bEdge\//.test(d.navigator&&d.navigator.userAgent);return{log:e("log"),info:e("info"),warn:e("warn"),error:e("error"),debug:function(){var c=e("debug");return function(){a&&c.apply(b,arguments)}}()}}]}function Cf(a){return a+""}function Df(a,b){return"undefined"!==typeof a?a:b}function Cd(a,b){return"undefined"===
typeof a?b:"undefined"===typeof b?a:a+b}function Ef(a,b){switch(a.type){case q.MemberExpression:if(a.computed)return!1;break;case q.UnaryExpression:return 1;case q.BinaryExpression:return"+"!==a.operator?1:!1;case q.CallExpression:return!1}return void 0===b?Dd:b}function Z(a,b,d){var c=a.isPure=Ef(a,d);switch(a.type){case q.Program:var e=!0;p(a.body,function(a){Z(a.expression,b,c);e=e&&a.expression.constant});a.constant=e;break;case q.Literal:a.constant=!0;a.toWatch=[];break;case q.UnaryExpression:Z(a.argument,
b,c);a.constant=a.argument.constant;a.toWatch=a.argument.toWatch;break;case q.BinaryExpression:Z(a.left,b,c);Z(a.right,b,c);a.constant=a.left.constant&&a.right.constant;a.toWatch=a.left.toWatch.concat(a.right.toWatch);break;case q.LogicalExpression:Z(a.left,b,c);Z(a.right,b,c);a.constant=a.left.constant&&a.right.constant;a.toWatch=a.constant?[]:[a];break;case q.ConditionalExpression:Z(a.test,b,c);Z(a.alternate,b,c);Z(a.consequent,b,c);a.constant=a.test.constant&&a.alternate.constant&&a.consequent.constant;
a.toWatch=a.constant?[]:[a];break;case q.Identifier:a.constant=!1;a.toWatch=[a];break;case q.MemberExpression:Z(a.object,b,c);a.computed&&Z(a.property,b,c);a.constant=a.object.constant&&(!a.computed||a.property.constant);a.toWatch=a.constant?[]:[a];break;case q.CallExpression:e=d=a.filter?!b(a.callee.name).$stateful:!1;var g=[];p(a.arguments,function(a){Z(a,b,c);e=e&&a.constant;g.push.apply(g,a.toWatch)});a.constant=e;a.toWatch=d?g:[a];break;case q.AssignmentExpression:Z(a.left,b,c);Z(a.right,b,c);
a.constant=a.left.constant&&a.right.constant;a.toWatch=[a];break;case q.ArrayExpression:e=!0;g=[];p(a.elements,function(a){Z(a,b,c);e=e&&a.constant;g.push.apply(g,a.toWatch)});a.constant=e;a.toWatch=g;break;case q.ObjectExpression:e=!0;g=[];p(a.properties,function(a){Z(a.value,b,c);e=e&&a.value.constant;g.push.apply(g,a.value.toWatch);a.computed&&(Z(a.key,b,!1),e=e&&a.key.constant,g.push.apply(g,a.key.toWatch))});a.constant=e;a.toWatch=g;break;case q.ThisExpression:a.constant=!1;a.toWatch=[];break;
case q.LocalsExpression:a.constant=!1,a.toWatch=[]}}function Ed(a){if(1===a.length){a=a[0].expression;var b=a.toWatch;return 1!==b.length?b:b[0]!==a?b:void 0}}function Fd(a){return a.type===q.Identifier||a.type===q.MemberExpression}function Gd(a){if(1===a.body.length&&Fd(a.body[0].expression))return{type:q.AssignmentExpression,left:a.body[0].expression,right:{type:q.NGValueParameter},operator:"="}}function Hd(a){this.$filter=a}function Id(a){this.$filter=a}function Qb(a,b,d){this.ast=new q(a,d);this.astCompiler=
d.csp?new Id(b):new Hd(b)}function Hc(a){return A(a.valueOf)?a.valueOf():Ff.call(a)}function Gf(){var a=V(),b={"true":!0,"false":!1,"null":null,undefined:void 0},d,c;this.addLiteral=function(a,c){b[a]=c};this.setIdentifierFns=function(a,b){d=a;c=b;return this};this.$get=["$filter",function(e){function g(b,c){switch(typeof b){case "string":var d=b=b.trim();var f=a[d];f||(f=new Rb(t),f=(new Qb(f,e,t)).parse(b),f.constant?f.$$watchDelegate=m:f.oneTime?f.$$watchDelegate=f.literal?l:k:f.inputs&&(f.$$watchDelegate=
h),a[d]=f);return n(f,c);case "function":return n(b,c);default:return n(H,c)}}function f(a,b,c){return null==a||null==b?a===b:"object"!==typeof a||(a=Hc(a),"object"!==typeof a||c)?a===b||a!==a&&b!==b:!1}function h(a,b,c,d,e){var g=d.inputs,h;if(1===g.length){var k=f;g=g[0];return a.$watch(function(a){var b=g(a);f(b,k,g.isPure)||(h=d(a,void 0,void 0,[b]),k=b&&Hc(b));return h},b,c,e)}for(var l=[],w=[],n=0,m=g.length;n<m;n++)l[n]=f,w[n]=null;return a.$watch(function(a){for(var b=!1,c=0,e=g.length;c<
e;c++){var k=g[c](a);if(b||(b=!f(k,l[c],g[c].isPure)))w[c]=k,l[c]=k&&Hc(k)}b&&(h=d(a,void 0,void 0,w));return h},b,c,e)}function k(a,b,c,d,e){function f(a){return d(a)}function g(a,c,d){l=a;A(b)&&b(a,c,d);v(a)&&d.$$postDigest(function(){v(l)&&k()})}var k,l;return k=d.inputs?h(a,g,c,d,e):a.$watch(f,g,c)}function l(a,b,c,d){function e(a){var b=!0;p(a,function(a){v(a)||(b=!1)});return b}var f,g;return f=a.$watch(function(a){return d(a)},function(a,c,d){g=a;A(b)&&b(a,c,d);e(a)&&d.$$postDigest(function(){e(g)&&
f()})},c)}function m(a,b,c,d){var e=a.$watch(function(a){e();return d(a)},b,c);return e}function n(a,b){if(!b)return a;var c=a.$$watchDelegate,d=!1,e=c!==l&&c!==k?function(c,e,f,g){f=d&&g?g[0]:a(c,e,f,g);return b(f,c,e)}:function(c,d,e,f){e=a(c,d,e,f);c=b(e,c,d);return v(e)?c:e};d=!a.inputs;c&&c!==h?(e.$$watchDelegate=c,e.inputs=a.inputs):b.$stateful||(e.$$watchDelegate=h,e.inputs=a.inputs?a.inputs:[a]);e.inputs&&(e.inputs=e.inputs.map(function(a){return a.isPure===Dd?function(b){return a(b)}:a}));
return e}var t={csp:Ea().noUnsafeEval,literals:Ga(b),isIdentifierStart:A(d)&&d,isIdentifierContinue:A(c)&&c};g.$$getAst=function(a){var b=new Rb(t);return(new Qb(b,e,t)).getAst(a).ast};return g}]}function Hf(){var a=!0;this.$get=["$rootScope","$exceptionHandler",function(b,d){return Jd(function(a){b.$evalAsync(a)},d,a)}];this.errorOnUnhandledRejections=function(b){return v(b)?(a=b,this):a}}function If(){var a=!0;this.$get=["$browser","$exceptionHandler",function(b,d){return Jd(function(a){b.defer(a)},
d,a)}];this.errorOnUnhandledRejections=function(b){return v(b)?(a=b,this):a}}function Jd(a,b,d){function c(){return new e}function e(){var a=this.promise=new g;this.resolve=function(b){k(a,b)};this.reject=function(b){m(a,b)};this.notify=function(b){t(a,b)}}function g(){this.$$state={status:0}}function f(){for(;!O&&v.length;){var a=v.shift();if(!a.pur){a.pur=!0;var c=a.value;c="Possibly unhandled rejection: "+("function"===typeof c?c.toString().replace(/ \{[\s\S]*$/,""):z(c)?"undefined":"string"!==
typeof c?He(c,void 0):c);gc(a.value)?b(a.value,c):b(c)}}}function h(c){!d||c.pending||2!==c.status||c.pur||(0===O&&0===v.length&&a(f),v.push(c));!c.processScheduled&&c.pending&&(c.processScheduled=!0,++O,a(function(){var e=c.pending;c.processScheduled=!1;c.pending=void 0;try{for(var g=0,h=e.length;g<h;++g){c.pur=!0;var l=e[g][0];var n=e[g][c.status];try{A(n)?k(l,n(c.value)):1===c.status?k(l,c.value):m(l,c.value)}catch(Q){m(l,Q),Q&&!0===Q.$$passToExceptionHandler&&b(Q)}}}finally{--O,d&&0===O&&a(f)}}))}
function k(a,b){a.$$state.status||(b===a?n(a,r("qcycle",b)):l(a,b))}function l(a,b){function c(b){f||(f=!0,l(a,b))}function d(b){f||(f=!0,n(a,b))}function e(b){t(a,b)}var f=!1;try{if(G(b)||A(b))var g=b.then;A(g)?(a.$$state.status=-1,g.call(b,c,d,e)):(a.$$state.value=b,a.$$state.status=1,h(a.$$state))}catch(R){d(R)}}function m(a,b){a.$$state.status||n(a,b)}function n(a,b){a.$$state.value=b;a.$$state.status=2;h(a.$$state)}function t(c,d){var e=c.$$state.pending;0>=c.$$state.status&&e&&e.length&&a(function(){for(var a,
c,f=0,g=e.length;f<g;f++){c=e[f][0];a=e[f][3];try{t(c,A(a)?a(d):d)}catch(R){b(R)}}})}function E(a){var b=new g;m(b,a);return b}function B(a,b,c){var d=null;try{A(c)&&(d=c())}catch(Ka){return E(Ka)}return d&&A(d.then)?d.then(function(){return b(a)},E):b(a)}function u(a,b,c,d){var e=new g;k(e,a);return e.then(b,c,d)}function q(a){if(!A(a))throw r("norslvr",a);var b=new g;a(function(a){k(b,a)},function(a){m(b,a)});return b}var r=K("$q",TypeError),O=0,v=[];P(g.prototype,{then:function(a,b,c){if(z(a)&&
z(b)&&z(c))return this;var d=new g;this.$$state.pending=this.$$state.pending||[];this.$$state.pending.push([d,a,b,c]);0<this.$$state.status&&h(this.$$state);return d},"catch":function(a){return this.then(null,a)},"finally":function(a,b){return this.then(function(b){return B(b,y,a)},function(b){return B(b,E,a)},b)}});var y=u;q.prototype=g.prototype;q.defer=c;q.reject=E;q.when=u;q.resolve=y;q.all=function(a){var b=new g,c=0,d=J(a)?[]:{};p(a,function(a,e){c++;u(a).then(function(a){d[e]=a;--c||k(b,d)},
function(a){m(b,a)})});0===c&&k(b,d);return b};q.race=function(a){var b=c();p(a,function(a){u(a).then(b.resolve,b.reject)});return b.promise};return q}function Jf(){this.$get=["$window","$timeout",function(a,b){var d=a.requestAnimationFrame||a.webkitRequestAnimationFrame,c=a.cancelAnimationFrame||a.webkitCancelAnimationFrame||a.webkitCancelRequestAnimationFrame,e=(a=!!d)?function(a){var b=d(a);return function(){c(b)}}:function(a){var c=b(a,16.66,!1);return function(){b.cancel(c)}};e.supported=a;return e}]}
function Kf(){function a(a){function b(){this.$$watchers=this.$$nextSibling=this.$$childHead=this.$$childTail=null;this.$$listeners={};this.$$listenerCount={};this.$$watchersCount=0;this.$id=++wb;this.$$ChildScope=null;this.$$suspended=!1}b.prototype=a;return b}var b=10,d=K("$rootScope"),c=null,e=null;this.digestTtl=function(a){arguments.length&&(b=a);return b};this.$get=["$exceptionHandler","$parse","$browser",function(g,f,h){function k(a){a.currentScope.$$destroyed=!0}function l(a){9===La&&(a.$$childHead&&
l(a.$$childHead),a.$$nextSibling&&l(a.$$nextSibling));a.$parent=a.$$nextSibling=a.$$prevSibling=a.$$childHead=a.$$childTail=a.$root=a.$$watchers=null}function m(){this.$id=++wb;this.$$phase=this.$parent=this.$$watchers=this.$$nextSibling=this.$$prevSibling=this.$$childHead=this.$$childTail=null;this.$root=this;this.$$suspended=this.$$destroyed=!1;this.$$listeners={};this.$$listenerCount={};this.$$watchersCount=0;this.$$isolateBindings=null}function n(a){if(r.$$phase)throw d("inprog",r.$$phase);r.$$phase=
a}function t(a,b){do a.$$watchersCount+=b;while(a=a.$parent)}function E(a,b,c){do a.$$listenerCount[c]-=b,0===a.$$listenerCount[c]&&delete a.$$listenerCount[c];while(a=a.$parent)}function q(){}function u(){for(;y.length;)try{y.shift()()}catch(w){g(w)}e=null}function F(){null===e&&(e=h.defer(function(){r.$apply(u)}))}m.prototype={constructor:m,$new:function(b,c){c=c||this;if(b){var d=new m;d.$root=this.$root}else this.$$ChildScope||(this.$$ChildScope=a(this)),d=new this.$$ChildScope;d.$parent=c;d.$$prevSibling=
c.$$childTail;c.$$childHead?(c.$$childTail.$$nextSibling=d,c.$$childTail=d):c.$$childHead=c.$$childTail=d;(b||c!==this)&&d.$on("$destroy",k);return d},$watch:function(a,b,d,e){var g=f(a);b=A(b)?b:H;if(g.$$watchDelegate)return g.$$watchDelegate(this,b,d,g,a);var h=this,k=h.$$watchers,l={fn:b,last:q,get:g,exp:e||a,eq:!!d};c=null;k||(k=h.$$watchers=[],k.$$digestWatchIndex=-1);k.unshift(l);k.$$digestWatchIndex++;t(this,1);return function(){var a=eb(k,l);0<=a&&(t(h,-1),a<k.$$digestWatchIndex&&k.$$digestWatchIndex--);
c=null}},$watchGroup:function(a,b){function c(){h=!1;k?(k=!1,b(e,e,g)):b(e,d,g)}var d=Array(a.length),e=Array(a.length),f=[],g=this,h=!1,k=!0;if(!a.length){var l=!0;g.$evalAsync(function(){l&&b(e,e,g)});return function(){l=!1}}if(1===a.length)return this.$watch(a[0],function(a,c,f){e[0]=a;d[0]=c;b(e,a===c?e:d,f)});p(a,function(a,b){a=g.$watch(a,function(a,f){e[b]=a;d[b]=f;h||(h=!0,g.$evalAsync(c))});f.push(a)});return function(){for(;f.length;)f.shift()()}},$watchCollection:function(a,b){function c(a){e=
a;var b;if(!z(e)){if(G(e))if(pa(e))for(g!==n&&(g=n,r=g.length=0,l++),a=e.length,r!==a&&(l++,g.length=r=a),b=0;b<a;b++){var c=g[b];var d=e[b];var f=c!==c&&d!==d;f||c===d||(l++,g[b]=d)}else{g!==m&&(g=m={},r=0,l++);a=0;for(b in e)ua.call(e,b)&&(a++,d=e[b],c=g[b],b in g?(f=c!==c&&d!==d,f||c===d||(l++,g[b]=d)):(r++,g[b]=d,l++));if(r>a)for(b in l++,g)ua.call(e,b)||(r--,delete g[b])}else g!==e&&(g=e,l++);return l}}c.$stateful=!0;var d=this,e,g,h,k=1<b.length,l=0;a=f(a,c);var n=[],m={},t=!0,r=0;return this.$watch(a,
function(){t?(t=!1,b(e,e,d)):b(e,h,d);if(k)if(G(e))if(pa(e)){h=Array(e.length);for(var a=0;a<e.length;a++)h[a]=e[a]}else for(a in h={},e)ua.call(e,a)&&(h[a]=e[a]);else h=e})},$digest:function(){var a,f,k,l,m=b,t=[];n("$digest");h.$$checkUrlChange();this===r&&null!==e&&(h.defer.cancel(e),u());c=null;do{var E=!1;var p=this;for(l=0;l<v.length;l++){try{var y=v[l];var B=y.fn;B(y.scope,y.locals)}catch(ob){g(ob)}c=null}v.length=0;a:do{if(l=!p.$$suspended&&p.$$watchers)for(l.$$digestWatchIndex=l.length;l.$$digestWatchIndex--;)try{if(a=
l[l.$$digestWatchIndex]){var F=a.get;if((f=F(p))!==(k=a.last)&&!(a.eq?va(f,k):aa(f)&&aa(k))){if(E=!0,c=a,a.last=a.eq?Ga(f,null):f,B=a.fn,B(f,k===q?f:k,p),5>m){var O=4-m;t[O]||(t[O]=[]);t[O].push({msg:A(a.exp)?"fn: "+(a.exp.name||a.exp.toString()):a.exp,newVal:f,oldVal:k})}}else if(a===c){E=!1;break a}}}catch(ob){g(ob)}if(!(l=!p.$$suspended&&p.$$watchersCount&&p.$$childHead||p!==this&&p.$$nextSibling))for(;p!==this&&!(l=p.$$nextSibling);)p=p.$parent}while(p=l);if((E||v.length)&&!m--)throw r.$$phase=
null,d("infdig",b,t);}while(E||v.length);for(r.$$phase=null;C<x.length;)try{x[C++]()}catch(ob){g(ob)}x.length=C=0;h.$$checkUrlChange()},$suspend:function(){this.$$suspended=!0},$isSuspended:function(){return this.$$suspended},$resume:function(){this.$$suspended=!1},$destroy:function(){if(!this.$$destroyed){var a=this.$parent;this.$broadcast("$destroy");this.$$destroyed=!0;this===r&&h.$$applicationDestroyed();t(this,-this.$$watchersCount);for(var b in this.$$listenerCount)E(this,this.$$listenerCount[b],
b);a&&a.$$childHead===this&&(a.$$childHead=this.$$nextSibling);a&&a.$$childTail===this&&(a.$$childTail=this.$$prevSibling);this.$$prevSibling&&(this.$$prevSibling.$$nextSibling=this.$$nextSibling);this.$$nextSibling&&(this.$$nextSibling.$$prevSibling=this.$$prevSibling);this.$destroy=this.$digest=this.$apply=this.$evalAsync=this.$applyAsync=H;this.$on=this.$watch=this.$watchGroup=function(){return H};this.$$listeners={};this.$$nextSibling=null;l(this)}},$eval:function(a,b){return f(a)(this,b)},$evalAsync:function(a,
b){r.$$phase||v.length||h.defer(function(){v.length&&r.$digest()});v.push({scope:this,fn:f(a),locals:b})},$$postDigest:function(a){x.push(a)},$apply:function(a){try{n("$apply");try{return this.$eval(a)}finally{r.$$phase=null}}catch(W){g(W)}finally{try{r.$digest()}catch(W){throw g(W),W;}}},$applyAsync:function(a){function b(){c.$eval(a)}var c=this;a&&y.push(b);a=f(a);F()},$on:function(a,b){var c=this.$$listeners[a];c||(this.$$listeners[a]=c=[]);c.push(b);var d=this;do d.$$listenerCount[a]||(d.$$listenerCount[a]=
0),d.$$listenerCount[a]++;while(d=d.$parent);var e=this;return function(){var d=c.indexOf(b);-1!==d&&(delete c[d],E(e,1,a))}},$emit:function(a,b){var c=[],d=this,e=!1,f={name:a,targetScope:d,stopPropagation:function(){e=!0},preventDefault:function(){f.defaultPrevented=!0},defaultPrevented:!1},h=fb([f],arguments,1),k;do{var l=d.$$listeners[a]||c;f.currentScope=d;var n=0;for(k=l.length;n<k;n++)if(l[n])try{l[n].apply(null,h)}catch(nb){g(nb)}else l.splice(n,1),n--,k--;if(e)break;d=d.$parent}while(d);
f.currentScope=null;return f},$broadcast:function(a,b){var c=this,d=this,e={name:a,targetScope:this,preventDefault:function(){e.defaultPrevented=!0},defaultPrevented:!1};if(!this.$$listenerCount[a])return e;for(var f=fb([e],arguments,1),h,k;c=d;){e.currentScope=c;d=c.$$listeners[a]||[];h=0;for(k=d.length;h<k;h++)if(d[h])try{d[h].apply(null,f)}catch(Ma){g(Ma)}else d.splice(h,1),h--,k--;if(!(d=c.$$listenerCount[a]&&c.$$childHead||c!==this&&c.$$nextSibling))for(;c!==this&&!(d=c.$$nextSibling);)c=c.$parent}e.currentScope=
null;return e}};var r=new m,v=r.$$asyncQueue=[],x=r.$$postDigestQueue=[],y=r.$$applyAsyncQueue=[],C=0;return r}]}function Lf(){var a=/^\s*(https?|s?ftp|mailto|tel|file):/,b=/^\s*((https?|ftp|file|blob):|data:image\/)/;this.aHrefSanitizationWhitelist=function(b){return v(b)?(a=b,this):a};this.imgSrcSanitizationWhitelist=function(a){return v(a)?(b=a,this):b};this.$get=function(){return function(d,c){c=c?b:a;var e=ha(d&&d.trim()).href;return""===e||e.match(c)?d:"unsafe:"+e}}}function Mf(){this.$get=
["$window","$document",function(a,b){var d={},c=!((!a.nw||!a.nw.process)&&a.chrome&&(a.chrome.app&&a.chrome.app.runtime||!a.chrome.app&&a.chrome.runtime&&a.chrome.runtime.id))&&a.history&&a.history.pushState,e=parseInt((/android (\d+)/.exec(N((a.navigator||{}).userAgent))||[])[1],10);a=/Boxee/i.test((a.navigator||{}).userAgent);var g=b[0]||{};b=g.body&&g.body.style;var f=!1,h=!1;b&&(f=!!("transition"in b||"webkitTransition"in b),h=!!("animation"in b||"webkitAnimation"in b));return{history:!(!c||4>
e||a),hasEvent:function(a){if("input"===a&&La)return!1;if(z(d[a])){var b=g.createElement("div");d[a]="on"+a in b}return d[a]},csp:Ea(),transitions:f,animations:h,android:e}}]}function Nf(){this.$get=["$rootScope","$browser","$location",function(a,b,d){return{findBindings:function(a,b,d){a=a.getElementsByClassName("ng-binding");var c=[];p(a,function(a){var e=ja.element(a).data("$binding");e&&p(e,function(e){d?(new RegExp("(^|\\s)"+b.replace(/([-()[\]{}+?*.$^|,:#<!\\])/g,"\\$1").replace(/\x08/g,"\\x08")+
"(\\s|\\||$)")).test(e)&&c.push(a):-1!==e.indexOf(b)&&c.push(a)})});return c},findModels:function(a,b,d){for(var c=["ng-","data-ng-","ng\\:"],e=0;e<c.length;++e){var g=a.querySelectorAll("["+c[e]+"model"+(d?"=":"*=")+'"'+b+'"]');if(g.length)return g}},getLocation:function(){return d.url()},setLocation:function(b){b!==d.url()&&(d.url(b),a.$digest())},whenStable:function(a){b.notifyWhenNoOutstandingRequests(a)}}}]}function Of(){this.$get=["$rootScope","$browser","$q","$$q","$exceptionHandler",function(a,
b,d,c,e){function g(g,k,l){A(g)||(l=k,k=g,g=H);var h=ya.call(arguments,3),n=v(l)&&!l,t=(n?c:d).defer(),p=t.promise;var q=b.defer(function(){try{t.resolve(g.apply(null,h))}catch(u){t.reject(u),e(u)}finally{delete f[p.$$timeoutId]}n||a.$apply()},k);p.$$timeoutId=q;f[q]=t;return p}var f={};g.cancel=function(a){return a&&a.$$timeoutId in f?(f[a.$$timeoutId].promise.$$state.pur=!0,f[a.$$timeoutId].reject("canceled"),delete f[a.$$timeoutId],b.defer.cancel(a.$$timeoutId)):!1};return g}]}function ha(a){if(!I(a))return a;
La&&(ba.setAttribute("href",a),a=ba.href);ba.setAttribute("href",a);return{href:ba.href,protocol:ba.protocol?ba.protocol.replace(/:$/,""):"",host:ba.host,search:ba.search?ba.search.replace(/^\?/,""):"",hash:ba.hash?ba.hash.replace(/^#/,""):"",hostname:ba.hostname,port:ba.port,pathname:"/"===ba.pathname.charAt(0)?ba.pathname:"/"+ba.pathname}}function sf(a){var b=[Kd].concat(a.map(ha));return function(a){a=ha(a);return b.some(Ld.bind(null,a))}}function Ld(a,b){a=ha(a);b=ha(b);return a.protocol===b.protocol&&
a.host===b.host}function Pf(){this.$get=la(D)}function Md(a){function b(a){try{return decodeURIComponent(a)}catch(f){return a}}var d=a[0]||{},c={},e="";return function(){var a;try{var f=d.cookie||""}catch(m){f=""}if(f!==e)for(e=f,f=e.split("; "),c={},a=0;a<f.length;a++){var h=f[a];var k=h.indexOf("=");if(0<k){var l=b(h.substring(0,k));z(c[l])&&(c[l]=b(h.substring(k+1)))}}return c}}function Qf(){this.$get=Md}function Nd(a){function b(d,c){if(G(d)){var e={};p(d,function(a,c){e[c]=b(c,a)});return e}return a.factory(d+
"Filter",c)}this.register=b;this.$get=["$injector",function(a){return function(b){return a.get(b+"Filter")}}];b("currency",Od);b("date",Pd);b("filter",Rf);b("json",Sf);b("limitTo",Tf);b("lowercase",Uf);b("number",Qd);b("orderBy",Rd);b("uppercase",Vf)}function Rf(){return function(a,b,d,c){if(!pa(a)){if(null==a)return a;throw K("filter")("notarray",a);}c=c||"$";switch(Ic(b)){case "function":break;case "boolean":case "null":case "number":case "string":var e=!0;case "object":b=Wf(b,d,c,e);break;default:return a}return Array.prototype.filter.call(a,
b)}}function Wf(a,b,d,c){var e=G(a)&&d in a;!0===b?b=va:A(b)||(b=function(a,b){if(z(a))return!1;if(null===a||null===b)return a===b;if(G(b)||G(a)&&!fc(a))return!1;a=N(""+a);b=N(""+b);return-1!==a.indexOf(b)});return function(g){return e&&!G(g)?Oa(g,a[d],b,d,!1):Oa(g,a,b,d,c)}}function Oa(a,b,d,c,e,g){var f=Ic(a),h=Ic(b);if("string"===h&&"!"===b.charAt(0))return!Oa(a,b.substring(1),d,c,e);if(J(a))return a.some(function(a){return Oa(a,b,d,c,e)});switch(f){case "object":var k;if(e){for(k in a)if(k.charAt&&
"$"!==k.charAt(0)&&Oa(a[k],b,d,c,!0))return!0;return g?!1:Oa(a,b,d,c,!1)}if("object"===h){for(k in b)if(g=b[k],!A(g)&&!z(g)&&(f=k===c,!Oa(f?a:a[k],g,d,c,f,f)))return!1;return!0}return d(a,b);case "function":return!1;default:return d(a,b)}}function Ic(a){return null===a?"null":typeof a}function Od(a){var b=a.NUMBER_FORMATS;return function(a,c,e){z(c)&&(c=b.CURRENCY_SYM);z(e)&&(e=b.PATTERNS[1].maxFrac);var d=c?/\u00A4/g:/\s*\u00A4\s*/g;return null==a?a:Sd(a,b.PATTERNS[1],b.GROUP_SEP,b.DECIMAL_SEP,e).replace(d,
c)}}function Qd(a){var b=a.NUMBER_FORMATS;return function(a,c){return null==a?a:Sd(a,b.PATTERNS[0],b.GROUP_SEP,b.DECIMAL_SEP,c)}}function Xf(a){var b=0,d,c,e,g;-1<(d=a.indexOf(Td))&&(a=a.replace(Td,""));0<(c=a.search(/e/i))?(0>d&&(d=c),d+=+a.slice(c+1),a=a.substring(0,c)):0>d&&(d=a.length);for(c=0;a.charAt(c)===Jc;c++);if(c===(g=a.length)){var f=[0];d=1}else{for(g--;a.charAt(g)===Jc;)g--;d-=c;f=[];for(e=0;c<=g;c++,e++)f[e]=+a.charAt(c)}d>Ud&&(f=f.splice(0,Ud-1),b=d-1,d=1);return{d:f,e:b,i:d}}function Yf(a,
b,d,c){var e=a.d,g=e.length-a.i;b=z(b)?Math.min(Math.max(d,g),c):+b;d=b+a.i;c=e[d];if(0<d){e.splice(Math.max(a.i,d));for(var f=d;f<e.length;f++)e[f]=0}else for(g=Math.max(0,g),a.i=1,e.length=Math.max(1,d=b+1),e[0]=0,f=1;f<d;f++)e[f]=0;if(5<=c)if(0>d-1){for(c=0;c>d;c--)e.unshift(0),a.i++;e.unshift(1);a.i++}else e[d-1]++;for(;g<Math.max(0,b);g++)e.push(0);if(b=e.reduceRight(function(a,b,c,d){b+=a;d[c]=b%10;return Math.floor(b/10)},0))e.unshift(b),a.i++}function Sd(a,b,d,c,e){if(!I(a)&&!ca(a)||isNaN(a))return"";
var g=!isFinite(a),f=!1,h=Math.abs(a)+"",k="";if(g)k="\u221e";else{f=Xf(h);Yf(f,e,b.minFrac,b.maxFrac);k=f.d;h=f.i;e=f.e;g=[];for(f=k.reduce(function(a,b){return a&&!b},!0);0>h;)k.unshift(0),h++;0<h?g=k.splice(h,k.length):(g=k,k=[0]);h=[];for(k.length>=b.lgSize&&h.unshift(k.splice(-b.lgSize,k.length).join(""));k.length>b.gSize;)h.unshift(k.splice(-b.gSize,k.length).join(""));k.length&&h.unshift(k.join(""));k=h.join(d);g.length&&(k+=c+g.join(""));e&&(k+="e+"+e)}return 0>a&&!f?b.negPre+k+b.negSuf:b.posPre+
k+b.posSuf}function Sb(a,b,d,c){var e="";if(0>a||c&&0>=a)c?a=-a+1:(a=-a,e="-");for(a=""+a;a.length<b;)a=Jc+a;d&&(a=a.substr(a.length-b));return e+a}function da(a,b,d,c,e){d=d||0;return function(g){g=g["get"+a]();if(0<d||g>-d)g+=d;0===g&&-12===d&&(g=12);return Sb(g,b,c,e)}}function sb(a,b,d){return function(c,e){c=c["get"+a]();var g=Ob((d?"STANDALONE":"")+(b?"SHORT":"")+a);return e[g][c]}}function Vd(a){var b=(new Date(a,0,1)).getDay();return new Date(a,0,(4>=b?5:12)-b)}function Wd(a){return function(b){var d=
Vd(b.getFullYear());b=new Date(b.getFullYear(),b.getMonth(),b.getDate()+(4-b.getDay()));return Sb(1+Math.round((+b-+d)/6048E5),a)}}function Kc(a,b){return 0>=a.getFullYear()?b.ERAS[0]:b.ERAS[1]}function Pd(a){function b(a){var b;if(b=a.match(d)){a=new Date(0);var c=0,f=0,h=b[8]?a.setUTCFullYear:a.setFullYear,k=b[8]?a.setUTCHours:a.setHours;b[9]&&(c=parseInt(b[9]+b[10],10),f=parseInt(b[9]+b[11],10));h.call(a,parseInt(b[1],10),parseInt(b[2],10)-1,parseInt(b[3],10));c=parseInt(b[4]||0,10)-c;f=parseInt(b[5]||
0,10)-f;h=parseInt(b[6]||0,10);b=Math.round(1E3*parseFloat("0."+(b[7]||0)));k.call(a,c,f,h,b)}return a}var d=/^(\d{4})-?(\d\d)-?(\d\d)(?:T(\d\d)(?::?(\d\d)(?::?(\d\d)(?:\.(\d+))?)?)?(Z|([+-])(\d\d):?(\d\d))?)?$/;return function(c,d,g){var e="",h=[],k,l;d=d||"mediumDate";d=a.DATETIME_FORMATS[d]||d;I(c)&&(c=Zf.test(c)?parseInt(c,10):b(c));ca(c)&&(c=new Date(c));if(!ia(c)||!isFinite(c.getTime()))return c;for(;d;)(l=$f.exec(d))?(h=fb(h,l,1),d=h.pop()):(h.push(d),d=null);var m=c.getTimezoneOffset();g&&
(m=ic(g,m),c=jc(c,g,!0));p(h,function(b){k=ag[b];e+=k?k(c,a.DATETIME_FORMATS,m):"''"===b?"'":b.replace(/(^'|'$)/g,"").replace(/''/g,"'")});return e}}function Sf(){return function(a,b){z(b)&&(b=2);return gb(a,b)}}function Tf(){return function(a,b,d){b=Infinity===Math.abs(Number(b))?Number(b):parseInt(b,10);if(aa(b))return a;ca(a)&&(a=a.toString());if(!pa(a))return a;d=!d||isNaN(d)?0:parseInt(d,10);d=0>d?Math.max(0,a.length+d):d;return 0<=b?Lc(a,d,d+b):0===d?Lc(a,b,a.length):Lc(a,Math.max(0,d+b),d)}}
function Lc(a,b,d){return I(a)?a.slice(b,d):ya.call(a,b,d)}function Rd(a){function b(b){return b.map(function(b){var c=1,d=yb;if(A(b))d=b;else if(I(b)){if("+"===b.charAt(0)||"-"===b.charAt(0))c="-"===b.charAt(0)?-1:1,b=b.substring(1);if(""!==b&&(d=a(b),d.constant)){var e=d();d=function(a){return a[e]}}}return{get:d,descending:c}})}function d(a){switch(typeof a){case "number":case "boolean":case "string":return!0;default:return!1}}function c(a,b){var c=0,d=a.type,e=b.type;if(d===e){e=a.value;var g=
b.value;"string"===d?(e=e.toLowerCase(),g=g.toLowerCase()):"object"===d&&(G(e)&&(e=a.index),G(g)&&(g=b.index));e!==g&&(c=e<g?-1:1)}else c=d<e?-1:1;return c}return function(a,g,f,h){if(null==a)return a;if(!pa(a))throw K("orderBy")("notarray",a);J(g)||(g=[g]);0===g.length&&(g=["+"]);var e=b(g),l=f?-1:1,m=A(h)?h:c;a=Array.prototype.map.call(a,function(a,b){return{value:a,tieBreaker:{value:b,type:"number",index:b},predicateValues:e.map(function(c){var e=c.get(a);c=typeof e;if(null===e)c="string",e="null";
else if("object"===c)a:{if(A(e.valueOf)&&(e=e.valueOf(),d(e)))break a;fc(e)&&(e=e.toString(),d(e))}return{value:e,type:c,index:b}})}});a.sort(function(a,b){for(var d=0,f=e.length;d<f;d++){var g=m(a.predicateValues[d],b.predicateValues[d]);if(g)return g*e[d].descending*l}return(m(a.tieBreaker,b.tieBreaker)||c(a.tieBreaker,b.tieBreaker))*l});return a=a.map(function(a){return a.value})}}function Pa(a){A(a)&&(a={link:a});a.restrict=a.restrict||"AC";return la(a)}function Tb(a,b,d,c,e){this.$$controls=
[];this.$error={};this.$$success={};this.$pending=void 0;this.$name=e(b.name||b.ngForm||"")(d);this.$dirty=!1;this.$valid=this.$pristine=!0;this.$submitted=this.$invalid=!1;this.$$parentForm=Ub;this.$$element=a;this.$$animate=c;Xd(this)}function Xd(a){a.$$classCache={};a.$$classCache[Yd]=!(a.$$classCache[tb]=a.$$element.hasClass(tb))}function Zd(a){function b(a,b,c){c&&!a.$$classCache[b]?(a.$$animate.addClass(a.$$element,b),a.$$classCache[b]=!0):!c&&a.$$classCache[b]&&(a.$$animate.removeClass(a.$$element,
b),a.$$classCache[b]=!1)}function d(a,c,d){c=c?"-"+cd(c,"-"):"";b(a,tb+c,!0===d);b(a,Yd+c,!1===d)}var c=a.set,e=a.unset;a.clazz.prototype.$setValidity=function(a,f,h){z(f)?(this.$pending||(this.$pending={}),c(this.$pending,a,h)):(this.$pending&&e(this.$pending,a,h),$d(this.$pending)&&(this.$pending=void 0));Ra(f)?f?(e(this.$error,a,h),c(this.$$success,a,h)):(c(this.$error,a,h),e(this.$$success,a,h)):(e(this.$error,a,h),e(this.$$success,a,h));this.$pending?(b(this,"ng-pending",!0),this.$valid=this.$invalid=
void 0,d(this,"",null)):(b(this,"ng-pending",!1),this.$valid=$d(this.$error),this.$invalid=!this.$valid,d(this,"",this.$valid));f=this.$pending&&this.$pending[a]?void 0:this.$error[a]?!1:this.$$success[a]?!0:null;d(this,a,f);this.$$parentForm.$setValidity(a,f,this)}}function $d(a){if(a)for(var b in a)if(a.hasOwnProperty(b))return!1;return!0}function Mc(a){a.$formatters.push(function(b){return a.$isEmpty(b)?b:b.toString()})}function Ua(a,b,d,c,e,g){var f=N(b[0].type);if(!e.android){var h=!1;b.on("compositionstart",
function(){h=!0});b.on("compositionupdate",function(a){if(z(a.data)||""===a.data)h=!1});b.on("compositionend",function(){h=!1;l()})}var k,l=function(a){k&&(g.defer.cancel(k),k=null);if(!h){var e=b.val();a=a&&a.type;"password"===f||d.ngTrim&&"false"===d.ngTrim||(e=T(e));(c.$viewValue!==e||""===e&&c.$$hasNativeValidators)&&c.$setViewValue(e,a)}};if(e.hasEvent("input"))b.on("input",l);else{var m=function(a,b,c){k||(k=g.defer(function(){k=null;b&&b.value===c||l(a)}))};b.on("keydown",function(a){var b=
a.keyCode;91===b||15<b&&19>b||37<=b&&40>=b||m(a,this,this.value)});if(e.hasEvent("paste"))b.on("paste cut drop",m)}b.on("change",l);if(ae[f]&&c.$$hasNativeValidators&&f===d.type)b.on("keydown wheel mousedown",function(a){if(!k){var b=this.validity,c=b.badInput,d=b.typeMismatch;k=g.defer(function(){k=null;b.badInput===c&&b.typeMismatch===d||l(a)})}});c.$render=function(){var a=c.$isEmpty(c.$viewValue)?"":c.$viewValue;b.val()!==a&&b.val(a)}}function Vb(a,b){return function(d,c){if(ia(d))return d;if(I(d)){'"'===
d.charAt(0)&&'"'===d.charAt(d.length-1)&&(d=d.substring(1,d.length-1));if(bg.test(d))return new Date(d);a.lastIndex=0;if(d=a.exec(d)){d.shift();var e=c?{yyyy:c.getFullYear(),MM:c.getMonth()+1,dd:c.getDate(),HH:c.getHours(),mm:c.getMinutes(),ss:c.getSeconds(),sss:c.getMilliseconds()/1E3}:{yyyy:1970,MM:1,dd:1,HH:0,mm:0,ss:0,sss:0};p(d,function(a,c){c<b.length&&(e[b[c]]=+a)});c=new Date(e.yyyy,e.MM-1,e.dd,e.HH,e.mm,e.ss||0,1E3*e.sss||0);100>e.yyyy&&c.setFullYear(e.yyyy);return c}}return NaN}}function ub(a,
b,d,c){return function(e,g,f,h,k,l,m){function n(a){return a&&!(a.getTime&&a.getTime()!==a.getTime())}function t(a){return v(a)&&!ia(a)?p(a)||void 0:a}function p(a,b){var c=h.$options.getOption("timezone");u&&u!==c&&(b=$c(b,ic(u)));a=d(a,b);!isNaN(a)&&c&&(a=jc(a,c));return a}Nc(e,g,f,h);Ua(e,g,f,h,k,l);var q,u;h.$$parserName=a;h.$parsers.push(function(a){if(h.$isEmpty(a))return null;if(b.test(a))return p(a,q)});h.$formatters.push(function(a){if(a&&!ia(a))throw vb("datefmt",a);if(n(a)){q=a;var b=h.$options.getOption("timezone");
b&&(u=b,q=jc(q,b,!0));return m("date")(a,c,b)}u=q=null;return""});if(v(f.min)||f.ngMin){var F;h.$validators.min=function(a){return!n(a)||z(F)||d(a)>=F};f.$observe("min",function(a){F=t(a);h.$validate()})}if(v(f.max)||f.ngMax){var r;h.$validators.max=function(a){return!n(a)||z(r)||d(a)<=r};f.$observe("max",function(a){r=t(a);h.$validate()})}}}function Nc(a,b,d,c){(c.$$hasNativeValidators=G(b[0].validity))&&c.$parsers.push(function(a){var c=b.prop("validity")||{};return c.badInput||c.typeMismatch?void 0:
a})}function be(a){a.$$parserName="number";a.$parsers.push(function(b){if(a.$isEmpty(b))return null;if(cg.test(b))return parseFloat(b)});a.$formatters.push(function(b){if(!a.$isEmpty(b)){if(!ca(b))throw vb("numfmt",b);b=b.toString()}return b})}function Va(a){v(a)&&!ca(a)&&(a=parseFloat(a));return aa(a)?void 0:a}function Oc(a){var b=a.toString(),d=b.indexOf(".");return-1===d?-1<a&&1>a&&(a=/e-(\d+)$/.exec(b))?Number(a[1]):0:b.length-d-1}function ce(a,b,d){a=Number(a);var c=(a|0)!==a,e=(b|0)!==b,g=(d|
0)!==d;if(c||e||g){var f=c?Oc(a):0,h=e?Oc(b):0,k=g?Oc(d):0;f=Math.pow(10,Math.max(f,h,k));a*=f;b*=f;d*=f;c&&(a=Math.round(a));e&&(b=Math.round(b));g&&(d=Math.round(d))}return 0===(a-b)%d}function de(a,b,d,c,e){if(v(c)){a=a(c);if(!a.constant)throw vb("constexpr",d,c);return a(b)}return e}function Pc(a,b){function d(a,b){if(!a||!a.length)return[];if(!b||!b.length)return a;var c=[],d=0;a:for(;d<a.length;d++){for(var e=a[d],f=0;f<b.length;f++)if(e===b[f])continue a;c.push(e)}return c}function c(a){var b=
a;J(a)?b=a.map(c).join(" "):G(a)&&(b=Object.keys(a).filter(function(b){return a[b]}).join(" "));return b}function e(a){var b=a;if(J(a))b=a.map(e);else if(G(a)){var c=!1;b=Object.keys(a).filter(function(b){b=a[b];!c&&z(b)&&(c=!0);return b});c&&b.push(void 0)}return b}a="ngClass"+a;var g;return["$parse",function(f){return{restrict:"AC",link:function(h,k,l){function m(a,b){var c=[];p(a,function(a){if(0<b||r[a])r[a]=(r[a]||0)+b,r[a]===+(0<b)&&c.push(a)});return c.join(" ")}function n(a){if(a===b){var c=
x;c=m(c&&c.split(" "),1);l.$addClass(c)}else c=x,c=m(c&&c.split(" "),-1),l.$removeClass(c);v=a}function t(a){a=c(a);a!==x&&q(a)}function q(a){if(v===b){var c=x&&x.split(" "),e=a&&a.split(" "),f=d(c,e);c=d(e,c);f=m(f,-1);c=m(c,1);l.$addClass(c);l.$removeClass(f)}x=a}var B=l[a].trim(),u=":"===B.charAt(0)&&":"===B.charAt(1);B=f(B,u?e:c);var F=u?t:q,r=k.data("$classCounts"),v=!0,x;r||(r=V(),k.data("$classCounts",r));"ngClass"!==a&&(g||(g=f("$index",function(a){return a&1})),h.$watch(g,n));h.$watch(B,
F,u)}}}]}function Wb(a,b,d,c,e,g,f,h,k){this.$modelValue=this.$viewValue=Number.NaN;this.$$rawModelValue=void 0;this.$validators={};this.$asyncValidators={};this.$parsers=[];this.$formatters=[];this.$viewChangeListeners=[];this.$untouched=!0;this.$touched=!1;this.$pristine=!0;this.$dirty=!1;this.$valid=!0;this.$invalid=!1;this.$error={};this.$$success={};this.$pending=void 0;this.$name=k(d.name||"",!1)(a);this.$$parentForm=Ub;this.$options=Qc;this.$$updateEvents="";this.$$updateEventHandler=this.$$updateEventHandler.bind(this);
this.$$parsedNgModel=e(d.ngModel);this.$$parsedNgModelAssign=this.$$parsedNgModel.assign;this.$$ngModelGet=this.$$parsedNgModel;this.$$ngModelSet=this.$$parsedNgModelAssign;this.$$pendingDebounce=null;this.$$parserValid=void 0;this.$$currentValidationRunId=0;Object.defineProperty(this,"$$scope",{value:a});this.$$attr=d;this.$$element=c;this.$$animate=g;this.$$timeout=f;this.$$parse=e;this.$$q=h;this.$$exceptionHandler=b;Xd(this);dg(this)}function dg(a){a.$$scope.$watch(function(b){b=a.$$ngModelGet(b);
b===a.$modelValue||a.$modelValue!==a.$modelValue&&b!==b||a.$$setModelValue(b);return b})}function Rc(a){this.$$options=a}function ee(a,b){p(b,function(b,c){v(a[c])||(a[c]=b)})}function Qa(a,b){a.prop("selected",b);a.attr("selected",b)}function eg(){this.SCE_CONTEXTS=$a;this.resourceUrlWhitelist=function(a){throw fa("noresourceurlwhitelist");};this.resourceUrlBlacklist=function(a){throw fa("noresourceurlblacklist");};this.$get=["$injector",function(a){var b=function(a){throw fa("unsafe");};a.has("$sanitize")&&
(b=a.get("$sanitize"));return{trustAs:function(a,b){throw fa("notrustas");},getTrusted:function(d,c){if(null===c||z(c)||""===c)return c;if("string"==typeof c){if(d==$a.TEMPLATE_URL){d=a.has("html2JsTemplatesCached")?!a.get("html2JsTemplatesCached")():!ng.safehtml.googSceHelper.isCOMPILED();if(d&&Ld(c,Kd))return c;throw fa("insecurl",c);}if(d==$a.RESOURCE_URL)throw fa("insecurl",c);if(d==$a.HTML)return b(c);throw fa("unsafe",d);}if(ng.safehtml.googSceHelper.isGoogHtmlType(c))try{return ng.safehtml.googSceHelper.unwrapGivenContext(d,
c)}catch(e){throw fa("googhtml",c,d);}else throw fa("unsafe",d);},valueOf:function(a){if(ng.safehtml.googSceHelper.isGoogHtmlType(a))try{return ng.safehtml.googSceHelper.unwrapAny(a)}catch(c){throw fa("googhtml",a);}else return a}}}]}function fg(){this.enabled=function(a){if(arguments.length)throw fa("nodisabling");return!0};this.$get=["$parse","$sceDelegate",function(a,b){if(8>La)throw fa("iequirks");if("undefined"==typeof ng||!ng.safehtml||!ng.safehtml.googSceHelper)throw fa("nodep");var d=ra($a);
d.isEnabled=function(){return!0};d.trustAs=b.trustAs;d.getTrusted=b.getTrusted;d.valueOf=b.valueOf;d.parseAs=function(b,c){var e=a(c);return e.literal&&e.constant?e:a(c,function(a){return d.getTrusted(b,a)})};var c=d.parseAs,e=d.getTrusted,g=d.trustAs;p($a,function(a,b){b=N(b);d[("parse_as_"+b).replace(Sc,Ba)]=function(b){return c(a,b)};d[("get_trusted_"+b).replace(Sc,Ba)]=function(b){return e(a,b)};d[("trust_as_"+b).replace(Sc,Ba)]=function(b){return g(a,b)}});return d}]}function gg(){var a;this.httpOptions=
function(b){return b?(a=b,this):a};this.$get=["$exceptionHandler","$templateCache","$http","$q","$sce",function(b,d,c,e,g){function f(h,k){f.totalPendingRequests++;if(!I(h)||z(d.get(h)))h=g.getTrustedTemplateUrl(h);var l=c.defaults&&c.defaults.transformResponse;J(l)?l=l.filter(function(a){return a!==Bc}):l===Bc&&(l=null);return c.get(h,P({cache:d,transformResponse:l},a)).finally(function(){f.totalPendingRequests--}).then(function(a){d.put(h,a.data);return a.data},function(a){k||(a=hg("tpload",h,a.status,
a.statusText),b(a));return e.reject(a)})}f.totalPendingRequests=0;return f}]}var Uc={objectMaxDepth:5},ig=/^\/(.+)\/([a-z]*)$/,ua=Object.prototype.hasOwnProperty,N=function(a){return I(a)?a.toLowerCase():a},Ob=function(a){return I(a)?a.toUpperCase():a},x,Fa,ya=[].slice,gf=[].splice,jg=[].push,ma=Object.prototype.toString,Xc=Object.getPrototypeOf,qa=K("ng"),ja=D.angular||(D.angular={}),uc,wb=0;var La=D.document.documentMode;var aa=Number.isNaN||function(a){return a!==a};H.$inject=[];yb.$inject=[];
var J=Array.isArray,xe=/^\[object (?:Uint8|Uint8Clamped|Uint16|Uint32|Int8|Int16|Int32|Float32|Float64)Array]$/,T=function(a){return I(a)?a.trim():a},Ea=function(){if(!v(Ea.rules)){var a=D.document.querySelector("[ng-csp]")||D.document.querySelector("[data-ng-csp]");if(a){var b=a.getAttribute("ng-csp")||a.getAttribute("data-ng-csp");Ea.rules={noUnsafeEval:!b||-1!==b.indexOf("no-unsafe-eval"),noInlineStyle:!b||-1!==b.indexOf("no-inline-style")}}else{a=Ea;try{new Function(""),b=!1}catch(d){b=!0}a.rules=
{noUnsafeEval:b,noInlineStyle:!1}}}return Ea.rules},Xb=function(){if(v(Xb.name_))return Xb.name_;var a,b,d=Ha.length;for(b=0;b<d;++b){var c=Ha[b];if(a=D.document.querySelector("["+c.replace(":","\\:")+"jq]")){var e=a.getAttribute(c+"jq");break}}return Xb.name_=e},ze=/:/g,Ha=["ng-","data-ng-","ng:","x-ng-"],Ce=function(a){var b=a.currentScript;if(!b)return!0;if(!(b instanceof D.HTMLScriptElement||b instanceof D.SVGScriptElement))return!1;b=b.attributes;return[b.getNamedItem("src"),b.getNamedItem("href"),
b.getNamedItem("xlink:href")].every(function(b){if(!b)return!0;if(!b.value)return!1;var c=a.createElement("a");c.href=b.value;if(a.location.origin===c.origin)return!0;switch(c.protocol){case "http:":case "https:":case "ftp:":case "blob:":case "file:":case "data:":return!0;default:return!1}})}(D.document),Fe=/[A-Z]/g,fe=!1,Sa=3,kg={full:"1.6.4-local+sha.617b36117",major:1,minor:6,dot:void 0,codeName:"undefined"};Y.expando="ng339";var lb=Y.cache={},Le=1;Y._data=function(a){return this.cache[a[this.expando]]||
{}};var Eb=/-([a-z])/g,lg=/^-ms-/,Db={mouseleave:"mouseout",mouseenter:"mouseover"},pc=K("jqLite"),Ke=/^<([\w-]+)\s*\/?>(?:<\/\1>|)$/,oc=/<|&#?\w+;/,Ie=/<([\w:-]+)/,Je=/<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:-]+)[^>]*)\/>/gi,oa={option:[1,'<select multiple="multiple">',"</select>"],thead:[1,"<table>","</table>"],col:[2,"<table><colgroup>","</colgroup></table>"],tr:[2,"<table><tbody>","</tbody></table>"],td:[3,"<table><tbody><tr>","</tr></tbody></table>"],_default:[0,"",""]};oa.optgroup=
oa.option;oa.tbody=oa.tfoot=oa.colgroup=oa.caption=oa.thead;oa.th=oa.td;var Qe=D.Node.prototype.contains||function(a){return!!(this.compareDocumentPosition(a)&16)},Za=Y.prototype={ready:fd,toString:function(){var a=[];p(this,function(b){a.push(""+b)});return"["+a.join(", ")+"]"},eq:function(a){return 0<=a?x(this[a]):x(this[this.length+a])},length:0,push:jg,sort:[].sort,splice:[].splice},Kb={};p("multiple selected checked disabled readOnly required open".split(" "),function(a){Kb[N(a)]=a});var kd=
{};p("input select option textarea button form details".split(" "),function(a){kd[a]=!0});var rd={ngMinlength:"minlength",ngMaxlength:"maxlength",ngMin:"min",ngMax:"max",ngPattern:"pattern",ngStep:"step"};p({data:tc,removeData:sc,hasData:function(a){for(var b in lb[a.ng339])return!0;return!1},cleanData:function(a){for(var b=0,d=a.length;b<d;b++)sc(a[b])}},function(a,b){Y[b]=a});p({data:tc,inheritedData:Ib,scope:function(a){return x.data(a,"$scope")||Ib(a.parentNode||a,["$isolateScope","$scope"])},
isolateScope:function(a){return x.data(a,"$isolateScope")||x.data(a,"$isolateScopeNoTemplate")},controller:hd,injector:function(a){return Ib(a,"$injector")},removeAttr:function(a,b){a.removeAttribute(b)},hasClass:Fb,css:function(a,b,d){b=b.replace(lg,"ms-").replace(Eb,Ba);if(v(d))a.style[b]=d;else return a.style[b]},attr:function(a,b,d){var c=a.nodeType;if(c!==Sa&&2!==c&&8!==c&&a.getAttribute){c=N(b);var e=Kb[c];if(v(d))null===d||!1===d&&e?a.removeAttribute(b):a.setAttribute(b,e?c:d);else return a=
a.getAttribute(b),e&&null!==a&&(a=c),null===a?void 0:a}},prop:function(a,b,d){if(v(d))a[b]=d;else return a[b]},text:function(){function a(a,d){if(z(d))return d=a.nodeType,1===d||d===Sa?a.textContent:"";a.textContent=d}a.$dv="";return a}(),val:function(a,b){if(z(b)){if(a.multiple&&"select"===za(a)){var d=[];p(a.options,function(a){a.selected&&d.push(a.value||a.text)});return d}return a.value}a.value=b},html:function(a,b){if(z(b))return a.innerHTML;Bb(a,!0);a.innerHTML=b},empty:id},function(a,b){Y.prototype[b]=
function(b,c){var d,g,f=this.length;if(a!==id&&z(2===a.length&&a!==Fb&&a!==hd?b:c)){if(G(b)){for(d=0;d<f;d++)if(a===tc)a(this[d],b);else for(g in b)a(this[d],g,b[g]);return this}d=a.$dv;f=z(d)?Math.min(f,1):f;for(g=0;g<f;g++){var h=a(this[g],b,c);d=d?d+h:h}return d}for(d=0;d<f;d++)a(this[d],b,c);return this}});p({removeData:sc,on:function(a,b,d,c){if(v(c))throw pc("onargs");if(nc(a)){c=Cb(a,!0);var e=c.events,g=c.handle;g||(g=c.handle=Ne(a,e));c=0<=b.indexOf(" ")?b.split(" "):[b];for(var f=c.length,
h=function(b,c,f){var h=e[b];h||(h=e[b]=[],h.specialHandlerWrapper=c,"$destroy"===b||f||a.addEventListener(b,g));h.push(d)};f--;)b=c[f],Db[b]?(h(Db[b],Pe),h(b,void 0,!0)):h(b)}},off:gd,one:function(a,b,d){a=x(a);a.on(b,function e(){a.off(b,d);a.off(b,e)});a.on(b,d)},replaceWith:function(a,b){var d,c=a.parentNode;Bb(a);p(new Y(b),function(b){d?c.insertBefore(b,d.nextSibling):c.replaceChild(b,a);d=b})},children:function(a){var b=[];p(a.childNodes,function(a){1===a.nodeType&&b.push(a)});return b},contents:function(a){return a.contentDocument||
a.childNodes||[]},append:function(a,b){var d=a.nodeType;if(1===d||11===d){b=new Y(b);d=0;for(var c=b.length;d<c;d++)a.appendChild(b[d])}},prepend:function(a,b){if(1===a.nodeType){var d=a.firstChild;p(new Y(b),function(b){a.insertBefore(b,d)})}},wrap:function(a,b){b=x(b).eq(0).clone()[0];var d=a.parentNode;d&&d.replaceChild(b,a);b.appendChild(a)},remove:Jb,detach:function(a){Jb(a,!0)},after:function(a,b){var d=a;if(a=a.parentNode){b=new Y(b);for(var c=0,e=b.length;c<e;c++){var g=b[c];a.insertBefore(g,
d.nextSibling);d=g}}},addClass:Hb,removeClass:Gb,toggleClass:function(a,b,d){b&&p(b.split(" "),function(b){var c=d;z(c)&&(c=!Fb(a,b));(c?Hb:Gb)(a,b)})},parent:function(a){return(a=a.parentNode)&&11!==a.nodeType?a:null},next:function(a){return a.nextElementSibling},find:function(a,b){return a.getElementsByTagName?a.getElementsByTagName(b):[]},clone:rc,triggerHandler:function(a,b,d){var c=b.type||b,e=Cb(a);if(e=(e=e&&e.events)&&e[c]){var g={preventDefault:function(){this.defaultPrevented=!0},isDefaultPrevented:function(){return!0===
this.defaultPrevented},stopImmediatePropagation:function(){this.immediatePropagationStopped=!0},isImmediatePropagationStopped:function(){return!0===this.immediatePropagationStopped},stopPropagation:H,type:c,target:a};b.type&&(g=P(g,b));b=ra(e);var f=d?[g].concat(d):[g];p(b,function(b){g.isImmediatePropagationStopped()||b.apply(a,f)})}}},function(a,b){Y.prototype[b]=function(b,c,e){for(var d,f=0,h=this.length;f<h;f++)z(d)?(d=a(this[f],b,c,e),v(d)&&(d=x(d))):qc(d,a(this[f],b,c,e));return v(d)?d:this}});
Y.prototype.bind=Y.prototype.on;Y.prototype.unbind=Y.prototype.off;var mg=Object.create(null);ld.prototype={_idx:function(a){if(a===this._lastKey)return this._lastIndex;this._lastKey=a;return this._lastIndex=this._keys.indexOf(a)},_transformKey:function(a){return aa(a)?mg:a},get:function(a){a=this._transformKey(a);a=this._idx(a);if(-1!==a)return this._values[a]},set:function(a,b){a=this._transformKey(a);var d=this._idx(a);-1===d&&(d=this._lastIndex=this._keys.length);this._keys[d]=a;this._values[d]=
b},delete:function(a){a=this._transformKey(a);a=this._idx(a);if(-1===a)return!1;this._keys.splice(a,1);this._values.splice(a,1);this._lastKey=NaN;this._lastIndex=-1;return!0}};var Lb=ld,og=[function(){this.$get=[function(){return Lb}]}],Te=/^([^(]+?)=>/,Ue=/^[^(]*\(\s*([^)]*)\)/m,pg=/,/,qg=/^\s*(_?)(\S+?)\1\s*$/,Se=/((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg,Ca=K("$injector");ib.$$annotate=function(a,b,d){var c;if("function"===typeof a){if(!(c=a.$inject)){c=[];if(a.length){if(b)throw I(d)&&d||(d=a.name||Ve(a)),
Ca("strictdi",d);b=md(a);p(b[1].split(pg),function(a){a.replace(qg,function(a,b,d){c.push(d)})})}a.$inject=c}}else J(a)?(b=a.length-1,zb(a[b],"fn"),c=a.slice(0,b)):zb(a,"fn",!0);return c};var ge=K("$animate"),rg=function(){this.$get=H},sg=function(){var a=new Lb,b=[];this.$get=["$$AnimateRunner","$rootScope",function(d,c){function e(a,b,c){var d=!1;b&&(b=I(b)?b.split(" "):J(b)?b:[],p(b,function(b){b&&(d=!0,a[b]=c)}));return d}function g(){p(b,function(b){var c=a.get(b);if(c){var d=Xe(b.attr("class")),
e="",f="";p(c,function(a,b){a!==!!d[b]&&(a?e+=(e.length?" ":"")+b:f+=(f.length?" ":"")+b)});p(b,function(a){e&&Hb(a,e);f&&Gb(a,f)});a.delete(b)}});b.length=0}return{enabled:H,on:H,off:H,pin:H,push:function(f,h,k,l){l&&l();k=k||{};k.from&&f.css(k.from);k.to&&f.css(k.to);if(k.addClass||k.removeClass)if(h=k.addClass,l=k.removeClass,k=a.get(f)||{},h=e(k,h,!0),l=e(k,l,!1),h||l)a.set(f,k),b.push(f),1===b.length&&c.$$postDigest(g);f=new d;f.complete();return f}}}]},tg=["$provide",function(a){var b=this,
d=null,c=null;this.$$registeredAnimations=Object.create(null);this.register=function(c,d){if(c&&"."!==c.charAt(0))throw ge("notcsel",c);var e=c+"-animation";b.$$registeredAnimations[c.substr(1)]=e;a.factory(e,d)};this.customFilter=function(a){1===arguments.length&&(c=A(a)?a:null);return c};this.classNameFilter=function(a){if(1===arguments.length&&(d=a instanceof RegExp?a:null)&&/[(\s|\/)]ng-animate[(\s|\/)]/.test(d.toString()))throw d=null,ge("nongcls","ng-animate");return d};this.$get=["$$animateQueue",
function(a){function b(a,b,c){if(c){var d;a:{for(d=0;d<c.length;d++){var e=c[d];if(1===e.nodeType){d=e;break a}}d=void 0}!d||d.parentNode||d.previousElementSibling||(c=null)}c?c.after(a):b.prepend(a)}return{on:a.on,off:a.off,pin:a.pin,enabled:a.enabled,cancel:function(a){a.end&&a.end()},enter:function(c,d,e,g){d=d&&x(d);e=e&&x(e);d=d||e.parent();b(c,d,e);return a.push(c,"enter",sa(g))},move:function(c,d,e,g){d=d&&x(d);e=e&&x(e);d=d||e.parent();b(c,d,e);return a.push(c,"move",sa(g))},leave:function(b,
c){return a.push(b,"leave",sa(c),function(){b.remove()})},addClass:function(b,c,d){d=sa(d);d.addClass=mb(d.addclass,c);return a.push(b,"addClass",d)},removeClass:function(b,c,d){d=sa(d);d.removeClass=mb(d.removeClass,c);return a.push(b,"removeClass",d)},setClass:function(b,c,d,e){e=sa(e);e.addClass=mb(e.addClass,c);e.removeClass=mb(e.removeClass,d);return a.push(b,"setClass",e)},animate:function(b,c,d,e,g){g=sa(g);g.from=g.from?P(g.from,c):c;g.to=g.to?P(g.to,d):d;g.tempClasses=mb(g.tempClasses,e||
"ng-inline-animate");return a.push(b,"animate",g)}}}]}],ug=function(){this.$get=["$$rAF",function(a){function b(b){d.push(b);1<d.length||a(function(){for(var a=0;a<d.length;a++)d[a]();d=[]})}var d=[];return function(){var a=!1;b(function(){a=!0});return function(c){a?c():b(c)}}}]},vg=function(){this.$get=["$q","$sniffer","$$animateAsyncRun","$$isDocumentHidden","$timeout",function(a,b,d,c,e){function g(a){this.setHost(a);var b=d();this._doneCallbacks=[];this._tick=function(a){c()?e(a,0,!1):b(a)};
this._state=0}g.chain=function(a,b){function c(){if(d===a.length)b(!0);else a[d](function(a){!1===a?b(!1):(d++,c())})}var d=0;c()};g.all=function(a,b){function c(c){e=e&&c;++d===a.length&&b(e)}var d=0,e=!0;p(a,function(a){a.done(c)})};g.prototype={setHost:function(a){this.host=a||{}},done:function(a){2===this._state?a():this._doneCallbacks.push(a)},progress:H,getPromise:function(){if(!this.promise){var b=this;this.promise=a(function(a,c){b.done(function(b){!1===b?c():a()})})}return this.promise},
then:function(a,b){return this.getPromise().then(a,b)},"catch":function(a){return this.getPromise()["catch"](a)},"finally":function(a){return this.getPromise()["finally"](a)},pause:function(){this.host.pause&&this.host.pause()},resume:function(){this.host.resume&&this.host.resume()},end:function(){this.host.end&&this.host.end();this._resolve(!0)},cancel:function(){this.host.cancel&&this.host.cancel();this._resolve(!1)},complete:function(a){var b=this;0===b._state&&(b._state=1,b._tick(function(){b._resolve(a)}))},
_resolve:function(a){2!==this._state&&(p(this._doneCallbacks,function(b){b(a)}),this._doneCallbacks.length=0,this._state=2)}};return g}]},wg=function(){this.$get=["$$rAF","$q","$$AnimateRunner",function(a,b,d){return function(b,e){function c(){a(function(){f.addClass&&(b.addClass(f.addClass),f.addClass=null);f.removeClass&&(b.removeClass(f.removeClass),f.removeClass=null);f.to&&(b.css(f.to),f.to=null);h||k.complete();h=!0});return k}var f=e||{};f.$$prepared||(f=Ga(f));f.cleanupStyles&&(f.from=f.to=
null);f.from&&(b.css(f.from),f.from=null);var h,k=new d;return{start:c,end:c}}}]},ea=K("$compile"),zc=new function(){};nd.$inject=["$provide","$$sanitizeUriProvider"];Mb.prototype.isFirstChange=function(){return this.previousValue===zc};var od=/^((?:x|data)[:\-_])/i,ff=/[:\-_]+(.)/g,td=K("$controller"),sd=/^(\S+)(\s+as\s+([\w$]+))?$/,xg=function(){this.$get=["$document",function(a){return function(b){b?!b.nodeType&&b instanceof x&&(b=b[0]):b=a[0].body;return b.offsetWidth+1}}]},ud="application/json",
Cc={"Content-Type":ud+";charset=utf-8"},pf=/^\[|^\{(?!\{)/,qf={"[":/]$/,"{":/}$/},of=/^\)]\}',?\n/,Nb=K("$http"),Na=ja.$interpolateMinErr=K("$interpolate");Na.throwNoconcat=function(a){throw Na("noconcat",a);};Na.interr=function(a,b){return Na("interr",a,b.toString())};var yg=function(){this.$get=function(){function a(a){var b=function(a){b.data=a;b.called=!0};b.id=a;return b}var b=ja.callbacks,d={};return{createCallback:function(c){c="_"+(b.$$counter++).toString(36);var e="angular.callbacks."+c,
g=a(c);d[e]=b[c]=g;return e},wasCalled:function(a){return d[a].called},getResponse:function(a){return d[a].data},removeCallback:function(a){delete b[d[a].id];delete d[a]}}}},zg=/^([^?#]*)(\?([^#]*))?(#(.*))?$/,yf={http:80,https:443,ftp:21},qb=K("$location"),zf=/^\s*[\\/]{2,}/,Ag={$$absUrl:"",$$html5:!1,$$replace:!1,absUrl:Pb("$$absUrl"),url:function(a){if(z(a))return this.$$url;var b=zg.exec(a);(b[1]||""===a)&&this.path(decodeURIComponent(b[1]));(b[2]||b[1]||""===a)&&this.search(b[3]||"");this.hash(b[5]||
"");return this},protocol:Pb("$$protocol"),host:Pb("$$host"),port:Pb("$$port"),path:Bd("$$path",function(a){a=null!==a?a.toString():"";return"/"===a.charAt(0)?a:"/"+a}),search:function(a,b){switch(arguments.length){case 0:return this.$$search;case 1:if(I(a)||ca(a))a=a.toString(),this.$$search=kc(a);else if(G(a))a=Ga(a,{}),p(a,function(b,c){null==b&&delete a[c]}),this.$$search=a;else throw qb("isrcharg");break;default:z(b)||null===b?delete this.$$search[a]:this.$$search[a]=b}this.$$compose();return this},
hash:Bd("$$hash",function(a){return null!==a?a.toString():""}),replace:function(){this.$$replace=!0;return this}};p([Ad,Gc,Fc],function(a){a.prototype=Object.create(Ag);a.prototype.state=function(b){if(!arguments.length)return this.$$state;if(a!==Fc||!this.$$html5)throw qb("nostate");this.$$state=z(b)?null:b;this.$$urlUpdatedByLocation=!0;return this}});var ab=K("$parse"),Ff={}.constructor.prototype.valueOf,Yb=V();p("+ - * / % === !== == != < > <= >= && || ! = |".split(" "),function(a){Yb[a]=!0});
var Bg={n:"\n",f:"\f",r:"\r",t:"\t",v:"\v","'":"'",'"':'"'},Rb=function(a){this.options=a};Rb.prototype={constructor:Rb,lex:function(a){this.text=a;this.index=0;for(this.tokens=[];this.index<this.text.length;)if(a=this.text.charAt(this.index),'"'===a||"'"===a)this.readString(a);else if(this.isNumber(a)||"."===a&&this.isNumber(this.peek()))this.readNumber();else if(this.isIdentifierStart(this.peekMultichar()))this.readIdent();else if(this.is(a,"(){}[].,;:?"))this.tokens.push({index:this.index,text:a}),
this.index++;else if(this.isWhitespace(a))this.index++;else{var b=a+this.peek(),d=b+this.peek(2),c=Yb[b],e=Yb[d];Yb[a]||c||e?(a=e?d:c?b:a,this.tokens.push({index:this.index,text:a,operator:!0}),this.index+=a.length):this.throwError("Unexpected next character ",this.index,this.index+1)}return this.tokens},is:function(a,b){return-1!==b.indexOf(a)},peek:function(a){a=a||1;return this.index+a<this.text.length?this.text.charAt(this.index+a):!1},isNumber:function(a){return"0"<=a&&"9">=a&&"string"===typeof a},
isWhitespace:function(a){return" "===a||"\r"===a||"\t"===a||"\n"===a||"\v"===a||"\u00a0"===a},isIdentifierStart:function(a){return this.options.isIdentifierStart?this.options.isIdentifierStart(a,this.codePointAt(a)):this.isValidIdentifierStart(a)},isValidIdentifierStart:function(a){return"a"<=a&&"z">=a||"A"<=a&&"Z">=a||"_"===a||"$"===a},isIdentifierContinue:function(a){return this.options.isIdentifierContinue?this.options.isIdentifierContinue(a,this.codePointAt(a)):this.isValidIdentifierContinue(a)},
isValidIdentifierContinue:function(a,b){return this.isValidIdentifierStart(a,b)||this.isNumber(a)},codePointAt:function(a){return 1===a.length?a.charCodeAt(0):(a.charCodeAt(0)<<10)+a.charCodeAt(1)-56613888},peekMultichar:function(){var a=this.text.charAt(this.index),b=this.peek();if(!b)return a;var d=a.charCodeAt(0),c=b.charCodeAt(0);return 55296<=d&&56319>=d&&56320<=c&&57343>=c?a+b:a},isExpOperator:function(a){return"-"===a||"+"===a||this.isNumber(a)},throwError:function(a,b,d){d=d||this.index;b=
v(b)?"s "+b+"-"+this.index+" ["+this.text.substring(b,d)+"]":" "+d;throw ab("lexerr",a,b,this.text);},readNumber:function(){for(var a="",b=this.index;this.index<this.text.length;){var d=N(this.text.charAt(this.index));if("."===d||this.isNumber(d))a+=d;else{var c=this.peek();if("e"===d&&this.isExpOperator(c))a+=d;else if(this.isExpOperator(d)&&c&&this.isNumber(c)&&"e"===a.charAt(a.length-1))a+=d;else if(!this.isExpOperator(d)||c&&this.isNumber(c)||"e"!==a.charAt(a.length-1))break;else this.throwError("Invalid exponent")}this.index++}this.tokens.push({index:b,
text:a,constant:!0,value:Number(a)})},readIdent:function(){var a=this.index;for(this.index+=this.peekMultichar().length;this.index<this.text.length;){var b=this.peekMultichar();if(!this.isIdentifierContinue(b))break;this.index+=b.length}this.tokens.push({index:a,text:this.text.slice(a,this.index),identifier:!0})},readString:function(a){var b=this.index;this.index++;for(var d="",c=a,e=!1;this.index<this.text.length;){var g=this.text.charAt(this.index);c+=g;if(e)"u"===g?(e=this.text.substring(this.index+
1,this.index+5),e.match(/[\da-f]{4}/i)||this.throwError("Invalid unicode escape [\\u"+e+"]"),this.index+=4,d+=String.fromCharCode(parseInt(e,16))):d+=Bg[g]||g,e=!1;else if("\\"===g)e=!0;else{if(g===a){this.index++;this.tokens.push({index:b,text:c,constant:!0,value:d});return}d+=g}this.index++}this.throwError("Unterminated quote",b)}};var q=function(a,b){this.lexer=a;this.options=b};q.Program="Program";q.ExpressionStatement="ExpressionStatement";q.AssignmentExpression="AssignmentExpression";q.ConditionalExpression=
"ConditionalExpression";q.LogicalExpression="LogicalExpression";q.BinaryExpression="BinaryExpression";q.UnaryExpression="UnaryExpression";q.CallExpression="CallExpression";q.MemberExpression="MemberExpression";q.Identifier="Identifier";q.Literal="Literal";q.ArrayExpression="ArrayExpression";q.Property="Property";q.ObjectExpression="ObjectExpression";q.ThisExpression="ThisExpression";q.LocalsExpression="LocalsExpression";q.NGValueParameter="NGValueParameter";q.prototype={ast:function(a){this.text=
a;this.tokens=this.lexer.lex(a);a=this.program();0!==this.tokens.length&&this.throwError("is an unexpected token",this.tokens[0]);return a},program:function(){for(var a=[];;)if(0<this.tokens.length&&!this.peek("}",")",";","]")&&a.push(this.expressionStatement()),!this.expect(";"))return{type:q.Program,body:a}},expressionStatement:function(){return{type:q.ExpressionStatement,expression:this.filterChain()}},filterChain:function(){for(var a=this.expression();this.expect("|");)a=this.filter(a);return a},
expression:function(){return this.assignment()},assignment:function(){var a=this.ternary();if(this.expect("=")){if(!Fd(a))throw ab("lval");a={type:q.AssignmentExpression,left:a,right:this.assignment(),operator:"="}}return a},ternary:function(){var a=this.logicalOR();if(this.expect("?")){var b=this.expression();if(this.consume(":")){var d=this.expression();return{type:q.ConditionalExpression,test:a,alternate:b,consequent:d}}}return a},logicalOR:function(){for(var a=this.logicalAND();this.expect("||");)a=
{type:q.LogicalExpression,operator:"||",left:a,right:this.logicalAND()};return a},logicalAND:function(){for(var a=this.equality();this.expect("&&");)a={type:q.LogicalExpression,operator:"&&",left:a,right:this.equality()};return a},equality:function(){for(var a=this.relational(),b;b=this.expect("==","!=","===","!==");)a={type:q.BinaryExpression,operator:b.text,left:a,right:this.relational()};return a},relational:function(){for(var a=this.additive(),b;b=this.expect("<",">","<=",">=");)a={type:q.BinaryExpression,
operator:b.text,left:a,right:this.additive()};return a},additive:function(){for(var a=this.multiplicative(),b;b=this.expect("+","-");)a={type:q.BinaryExpression,operator:b.text,left:a,right:this.multiplicative()};return a},multiplicative:function(){for(var a=this.unary(),b;b=this.expect("*","/","%");)a={type:q.BinaryExpression,operator:b.text,left:a,right:this.unary()};return a},unary:function(){var a;return(a=this.expect("+","-","!"))?{type:q.UnaryExpression,operator:a.text,prefix:!0,argument:this.unary()}:
this.primary()},primary:function(){if(this.expect("(")){var a=this.filterChain();this.consume(")")}else this.expect("[")?a=this.arrayDeclaration():this.expect("{")?a=this.object():this.selfReferential.hasOwnProperty(this.peek().text)?a=Ga(this.selfReferential[this.consume().text]):this.options.literals.hasOwnProperty(this.peek().text)?a={type:q.Literal,value:this.options.literals[this.consume().text]}:this.peek().identifier?a=this.identifier():this.peek().constant?a=this.constant():this.throwError("not a primary expression",
this.peek());for(var b;b=this.expect("(","[",".");)"("===b.text?(a={type:q.CallExpression,callee:a,arguments:this.parseArguments()},this.consume(")")):"["===b.text?(a={type:q.MemberExpression,object:a,property:this.expression(),computed:!0},this.consume("]")):"."===b.text?a={type:q.MemberExpression,object:a,property:this.identifier(),computed:!1}:this.throwError("IMPOSSIBLE");return a},filter:function(a){a=[a];for(var b={type:q.CallExpression,callee:this.identifier(),arguments:a,filter:!0};this.expect(":");)a.push(this.expression());
return b},parseArguments:function(){var a=[];if(")"!==this.peekToken().text){do a.push(this.filterChain());while(this.expect(","))}return a},identifier:function(){var a=this.consume();a.identifier||this.throwError("is not a valid identifier",a);return{type:q.Identifier,name:a.text}},constant:function(){return{type:q.Literal,value:this.consume().value}},arrayDeclaration:function(){var a=[];if("]"!==this.peekToken().text){do{if(this.peek("]"))break;a.push(this.expression())}while(this.expect(","))}this.consume("]");
return{type:q.ArrayExpression,elements:a}},object:function(){var a=[];if("}"!==this.peekToken().text){do{if(this.peek("}"))break;var b={type:q.Property,kind:"init"};this.peek().constant?(b.key=this.constant(),b.computed=!1,this.consume(":"),b.value=this.expression()):this.peek().identifier?(b.key=this.identifier(),b.computed=!1,this.peek(":")?(this.consume(":"),b.value=this.expression()):b.value=b.key):this.peek("[")?(this.consume("["),b.key=this.expression(),this.consume("]"),b.computed=!0,this.consume(":"),
b.value=this.expression()):this.throwError("invalid key",this.peek());a.push(b)}while(this.expect(","))}this.consume("}");return{type:q.ObjectExpression,properties:a}},throwError:function(a,b){throw ab("syntax",b.text,a,b.index+1,this.text,this.text.substring(b.index));},consume:function(a){if(0===this.tokens.length)throw ab("ueoe",this.text);var b=this.expect(a);b||this.throwError("is unexpected, expecting ["+a+"]",this.peek());return b},peekToken:function(){if(0===this.tokens.length)throw ab("ueoe",
this.text);return this.tokens[0]},peek:function(a,b,d,c){return this.peekAhead(0,a,b,d,c)},peekAhead:function(a,b,d,c,e){if(this.tokens.length>a){a=this.tokens[a];var g=a.text;if(g===b||g===d||g===c||g===e||!(b||d||c||e))return a}return!1},expect:function(a,b,d,c){return(a=this.peek(a,b,d,c))?(this.tokens.shift(),a):!1},selfReferential:{"this":{type:q.ThisExpression},$locals:{type:q.LocalsExpression}}};var Dd=2;Hd.prototype={compile:function(a){var b=this;this.state={nextId:0,filters:{},fn:{vars:[],
body:[],own:{}},assign:{vars:[],body:[],own:{}},inputs:[]};Z(a,b.$filter);var d="",c;this.stage="assign";if(c=Gd(a))this.state.computing="assign",d=this.nextId(),this.recurse(c,d),this.return_(d),d="fn.assign="+this.generateFunction("assign","s,v,l");c=Ed(a.body);b.stage="inputs";p(c,function(a,c){var d="fn"+c;b.state[d]={vars:[],body:[],own:{}};b.state.computing=d;var e=b.nextId();b.recurse(a,e);b.return_(e);b.state.inputs.push({name:d,isPure:a.isPure});a.watchId=c});this.state.computing="fn";this.stage=
"main";this.recurse(a);a='"'+this.USE+" "+this.STRICT+'";\n'+this.filterPrefix()+"var fn="+this.generateFunction("fn","s,l,a,i")+d+this.watchFns()+"return fn;";a=(new Function("$filter","getStringValue","ifDefined","plus",a))(this.$filter,Cf,Df,Cd);this.state=this.stage=void 0;return a},USE:"use",STRICT:"strict",watchFns:function(){var a=[],b=this.state.inputs,d=this;p(b,function(b){a.push("var "+b.name+"="+d.generateFunction(b.name,"s"));b.isPure&&a.push(b.name,".isPure="+JSON.stringify(b.isPure)+
";")});b.length&&a.push("fn.inputs=["+b.map(function(a){return a.name}).join(",")+"];");return a.join("")},generateFunction:function(a,b){return"function("+b+"){"+this.varsPrefix(a)+this.body(a)+"};"},filterPrefix:function(){var a=[],b=this;p(this.state.filters,function(d,c){a.push(d+"=$filter("+b.escape(c)+")")});return a.length?"var "+a.join(",")+";":""},varsPrefix:function(a){return this.state[a].vars.length?"var "+this.state[a].vars.join(",")+";":""},body:function(a){return this.state[a].body.join("")},
recurse:function(a,b,d,c,e,g){var f=this;c=c||H;if(!g&&v(a.watchId))b=b||this.nextId(),this.if_("i",this.lazyAssign(b,this.computedMember("i",a.watchId)),this.lazyRecurse(a,b,d,c,e,!0));else switch(a.type){case q.Program:p(a.body,function(b,c){f.recurse(b.expression,void 0,void 0,function(a){l=a});c!==a.body.length-1?f.current().body.push(l,";"):f.return_(l)});break;case q.Literal:var h=this.escape(a.value);this.assign(b,h);c(b||h);break;case q.UnaryExpression:this.recurse(a.argument,void 0,void 0,
function(a){l=a});h=a.operator+"("+this.ifDefined(l,0)+")";this.assign(b,h);c(h);break;case q.BinaryExpression:this.recurse(a.left,void 0,void 0,function(a){k=a});this.recurse(a.right,void 0,void 0,function(a){l=a});h="+"===a.operator?this.plus(k,l):"-"===a.operator?this.ifDefined(k,0)+a.operator+this.ifDefined(l,0):"("+k+")"+a.operator+"("+l+")";this.assign(b,h);c(h);break;case q.LogicalExpression:b=b||this.nextId();f.recurse(a.left,b);f.if_("&&"===a.operator?b:f.not(b),f.lazyRecurse(a.right,b));
c(b);break;case q.ConditionalExpression:b=b||this.nextId();f.recurse(a.test,b);f.if_(b,f.lazyRecurse(a.alternate,b),f.lazyRecurse(a.consequent,b));c(b);break;case q.Identifier:b=b||this.nextId();d&&(d.context="inputs"===f.stage?"s":this.assign(this.nextId(),this.getHasOwnProperty("l",a.name)+"?l:s"),d.computed=!1,d.name=a.name);f.if_("inputs"===f.stage||f.not(f.getHasOwnProperty("l",a.name)),function(){f.if_("inputs"===f.stage||"s",function(){e&&1!==e&&f.if_(f.isNull(f.nonComputedMember("s",a.name)),
f.lazyAssign(f.nonComputedMember("s",a.name),"{}"));f.assign(b,f.nonComputedMember("s",a.name))})},b&&f.lazyAssign(b,f.nonComputedMember("l",a.name)));c(b);break;case q.MemberExpression:var k=d&&(d.context=this.nextId())||this.nextId();b=b||this.nextId();f.recurse(a.object,k,void 0,function(){f.if_(f.notNull(k),function(){a.computed?(l=f.nextId(),f.recurse(a.property,l),f.getStringValue(l),e&&1!==e&&f.if_(f.not(f.computedMember(k,l)),f.lazyAssign(f.computedMember(k,l),"{}")),h=f.computedMember(k,
l),f.assign(b,h),d&&(d.computed=!0,d.name=l)):(e&&1!==e&&f.if_(f.isNull(f.nonComputedMember(k,a.property.name)),f.lazyAssign(f.nonComputedMember(k,a.property.name),"{}")),h=f.nonComputedMember(k,a.property.name),f.assign(b,h),d&&(d.computed=!1,d.name=a.property.name))},function(){f.assign(b,"undefined")});c(b)},!!e);break;case q.CallExpression:b=b||this.nextId();if(a.filter){var l=f.filter(a.callee.name);var m=[];p(a.arguments,function(a){var b=f.nextId();f.recurse(a,b);m.push(b)});h=l+"("+m.join(",")+
")";f.assign(b,h);c(b)}else l=f.nextId(),k={},m=[],f.recurse(a.callee,l,k,function(){f.if_(f.notNull(l),function(){p(a.arguments,function(b){f.recurse(b,a.constant?void 0:f.nextId(),void 0,function(a){m.push(a)})});h=k.name?f.member(k.context,k.name,k.computed)+"("+m.join(",")+")":l+"("+m.join(",")+")";f.assign(b,h)},function(){f.assign(b,"undefined")});c(b)});break;case q.AssignmentExpression:l=this.nextId();k={};this.recurse(a.left,void 0,k,function(){f.if_(f.notNull(k.context),function(){f.recurse(a.right,
l);h=f.member(k.context,k.name,k.computed)+a.operator+l;f.assign(b,h);c(b||h)})},1);break;case q.ArrayExpression:m=[];p(a.elements,function(b){f.recurse(b,a.constant?void 0:f.nextId(),void 0,function(a){m.push(a)})});h="["+m.join(",")+"]";this.assign(b,h);c(b||h);break;case q.ObjectExpression:m=[];var n=!1;p(a.properties,function(a){a.computed&&(n=!0)});n?(b=b||this.nextId(),this.assign(b,"{}"),p(a.properties,function(a){a.computed?(k=f.nextId(),f.recurse(a.key,k)):k=a.key.type===q.Identifier?a.key.name:
""+a.key.value;l=f.nextId();f.recurse(a.value,l);f.assign(f.member(b,k,a.computed),l)})):(p(a.properties,function(b){f.recurse(b.value,a.constant?void 0:f.nextId(),void 0,function(a){m.push(f.escape(b.key.type===q.Identifier?b.key.name:""+b.key.value)+":"+a)})}),h="{"+m.join(",")+"}",this.assign(b,h));c(b||h);break;case q.ThisExpression:this.assign(b,"s");c(b||"s");break;case q.LocalsExpression:this.assign(b,"l");c(b||"l");break;case q.NGValueParameter:this.assign(b,"v"),c(b||"v")}},getHasOwnProperty:function(a,
b){var d=a+"."+b,c=this.current().own;c.hasOwnProperty(d)||(c[d]=this.nextId(!1,a+"&&("+this.escape(b)+" in "+a+")"));return c[d]},assign:function(a,b){if(a)return this.current().body.push(a,"=",b,";"),a},filter:function(a){this.state.filters.hasOwnProperty(a)||(this.state.filters[a]=this.nextId(!0));return this.state.filters[a]},ifDefined:function(a,b){return"ifDefined("+a+","+this.escape(b)+")"},plus:function(a,b){return"plus("+a+","+b+")"},return_:function(a){this.current().body.push("return ",
a,";")},if_:function(a,b,d){if(!0===a)b();else{var c=this.current().body;c.push("if(",a,"){");b();c.push("}");d&&(c.push("else{"),d(),c.push("}"))}},not:function(a){return"!("+a+")"},isNull:function(a){return a+"==null"},notNull:function(a){return a+"!=null"},nonComputedMember:function(a,b){var d=/[^$_a-zA-Z0-9]/g;return/^[$_a-zA-Z][$_a-zA-Z0-9]*$/.test(b)?a+"."+b:a+'["'+b.replace(d,this.stringEscapeFn)+'"]'},computedMember:function(a,b){return a+"["+b+"]"},member:function(a,b,d){return d?this.computedMember(a,
b):this.nonComputedMember(a,b)},getStringValue:function(a){this.assign(a,"getStringValue("+a+")")},lazyRecurse:function(a,b,d,c,e,g){var f=this;return function(){f.recurse(a,b,d,c,e,g)}},lazyAssign:function(a,b){var d=this;return function(){d.assign(a,b)}},stringEscapeRegex:/[^ a-zA-Z0-9]/g,stringEscapeFn:function(a){return"\\u"+("0000"+a.charCodeAt(0).toString(16)).slice(-4)},escape:function(a){if(I(a))return"'"+a.replace(this.stringEscapeRegex,this.stringEscapeFn)+"'";if(ca(a))return a.toString();
if(!0===a)return"true";if(!1===a)return"false";if(null===a)return"null";if("undefined"===typeof a)return"undefined";throw ab("esc");},nextId:function(a,b){var d="v"+this.state.nextId++;a||this.current().vars.push(d+(b?"="+b:""));return d},current:function(){return this.state[this.state.computing]}};Id.prototype={compile:function(a){var b=this;Z(a,b.$filter);var d;if(d=Gd(a))var c=this.recurse(d);d=Ed(a.body);if(d){var e=[];p(d,function(a,c){var d=b.recurse(a);d.isPure=a.isPure;a.input=d;e.push(d);
a.watchId=c})}var g=[];p(a.body,function(a){g.push(b.recurse(a.expression))});a=0===a.body.length?H:1===a.body.length?g[0]:function(a,b){var c;p(g,function(d){c=d(a,b)});return c};c&&(a.assign=function(a,b,d){return c(a,d,b)});e&&(a.inputs=e);return a},recurse:function(a,b,d){var c=this;if(a.input)return this.inputs(a.input,a.watchId);switch(a.type){case q.Literal:return this.value(a.value,b);case q.UnaryExpression:var e=this.recurse(a.argument);return this["unary"+a.operator](e,b);case q.BinaryExpression:var g=
this.recurse(a.left);e=this.recurse(a.right);return this["binary"+a.operator](g,e,b);case q.LogicalExpression:return g=this.recurse(a.left),e=this.recurse(a.right),this["binary"+a.operator](g,e,b);case q.ConditionalExpression:return this["ternary?:"](this.recurse(a.test),this.recurse(a.alternate),this.recurse(a.consequent),b);case q.Identifier:return c.identifier(a.name,b,d);case q.MemberExpression:return g=this.recurse(a.object,!1,!!d),a.computed||(e=a.property.name),a.computed&&(e=this.recurse(a.property)),
a.computed?this.computedMember(g,e,b,d):this.nonComputedMember(g,e,b,d);case q.CallExpression:var f=[];p(a.arguments,function(a){f.push(c.recurse(a))});a.filter&&(e=this.$filter(a.callee.name));a.filter||(e=this.recurse(a.callee,!0));return a.filter?function(a,c,d,g){for(var h=[],k=0;k<f.length;++k)h.push(f[k](a,c,d,g));a=e.apply(void 0,h,g);return b?{context:void 0,name:void 0,value:a}:a}:function(a,c,d,g){var h=e(a,c,d,g);if(null!=h.value){var k=[];for(var l=0;l<f.length;++l)k.push(f[l](a,c,d,g));
k=h.value.apply(h.context,k)}return b?{value:k}:k};case q.AssignmentExpression:return g=this.recurse(a.left,!0,1),e=this.recurse(a.right),function(a,c,d,f){var h=g(a,c,d,f);a=e(a,c,d,f);h.context[h.name]=a;return b?{value:a}:a};case q.ArrayExpression:return f=[],p(a.elements,function(a){f.push(c.recurse(a))}),function(a,c,d,e){for(var g=[],h=0;h<f.length;++h)g.push(f[h](a,c,d,e));return b?{value:g}:g};case q.ObjectExpression:return f=[],p(a.properties,function(a){a.computed?f.push({key:c.recurse(a.key),
computed:!0,value:c.recurse(a.value)}):f.push({key:a.key.type===q.Identifier?a.key.name:""+a.key.value,computed:!1,value:c.recurse(a.value)})}),function(a,c,d,e){for(var g={},h=0;h<f.length;++h)f[h].computed?g[f[h].key(a,c,d,e)]=f[h].value(a,c,d,e):g[f[h].key]=f[h].value(a,c,d,e);return b?{value:g}:g};case q.ThisExpression:return function(a){return b?{value:a}:a};case q.LocalsExpression:return function(a,c){return b?{value:c}:c};case q.NGValueParameter:return function(a,c,d){return b?{value:d}:d}}},
"unary+":function(a,b){return function(d,c,e,g){d=a(d,c,e,g);d=v(d)?+d:0;return b?{value:d}:d}},"unary-":function(a,b){return function(d,c,e,g){d=a(d,c,e,g);d=v(d)?-d:-0;return b?{value:d}:d}},"unary!":function(a,b){return function(d,c,e,g){d=!a(d,c,e,g);return b?{value:d}:d}},"binary+":function(a,b,d){return function(c,e,g,f){var h=a(c,e,g,f);c=b(c,e,g,f);h=Cd(h,c);return d?{value:h}:h}},"binary-":function(a,b,d){return function(c,e,g,f){var h=a(c,e,g,f);c=b(c,e,g,f);h=(v(h)?h:0)-(v(c)?c:0);return d?
{value:h}:h}},"binary*":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)*b(c,e,g,f);return d?{value:c}:c}},"binary/":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)/b(c,e,g,f);return d?{value:c}:c}},"binary%":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)%b(c,e,g,f);return d?{value:c}:c}},"binary===":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)===b(c,e,g,f);return d?{value:c}:c}},"binary!==":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)!==b(c,e,g,f);return d?{value:c}:
c}},"binary==":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)==b(c,e,g,f);return d?{value:c}:c}},"binary!=":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)!=b(c,e,g,f);return d?{value:c}:c}},"binary<":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)<b(c,e,g,f);return d?{value:c}:c}},"binary>":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)>b(c,e,g,f);return d?{value:c}:c}},"binary<=":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)<=b(c,e,g,f);return d?{value:c}:c}},"binary>=":function(a,
b,d){return function(c,e,g,f){c=a(c,e,g,f)>=b(c,e,g,f);return d?{value:c}:c}},"binary&&":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)&&b(c,e,g,f);return d?{value:c}:c}},"binary||":function(a,b,d){return function(c,e,g,f){c=a(c,e,g,f)||b(c,e,g,f);return d?{value:c}:c}},"ternary?:":function(a,b,d,c){return function(e,g,f,h){e=a(e,g,f,h)?b(e,g,f,h):d(e,g,f,h);return c?{value:e}:e}},value:function(a,b){return function(){return b?{context:void 0,name:void 0,value:a}:a}},identifier:function(a,
b,d){return function(c,e,g,f){c=e&&a in e?e:c;d&&1!==d&&c&&null==c[a]&&(c[a]={});e=c?c[a]:void 0;return b?{context:c,name:a,value:e}:e}},computedMember:function(a,b,d,c){return function(e,g,f,h){var k=a(e,g,f,h);if(null!=k){var l=b(e,g,f,h);l+="";c&&1!==c&&k&&!k[l]&&(k[l]={});var m=k[l]}return d?{context:k,name:l,value:m}:m}},nonComputedMember:function(a,b,d,c){return function(e,g,f,h){e=a(e,g,f,h);c&&1!==c&&e&&null==e[b]&&(e[b]={});g=null!=e?e[b]:void 0;return d?{context:e,name:b,value:g}:g}},inputs:function(a,
b){return function(d,c,e,g){return g?g[b]:a(d,c,e)}}};Qb.prototype={constructor:Qb,parse:function(a){a=this.getAst(a);var b=this.astCompiler.compile(a.ast),d=a.ast;b.literal=0===d.body.length||1===d.body.length&&(d.body[0].expression.type===q.Literal||d.body[0].expression.type===q.ArrayExpression||d.body[0].expression.type===q.ObjectExpression);b.constant=a.ast.constant;b.oneTime=a.oneTime;return b},getAst:function(a){var b=!1;a=a.trim();":"===a.charAt(0)&&":"===a.charAt(1)&&(b=!0,a=a.substring(2));
return{ast:this.ast.ast(a),oneTime:b}}};var ba=D.document.createElement("a"),Kd=ha(D.location.href);Md.$inject=["$document"];Nd.$inject=["$provide"];var Ud=22,Td=".",Jc="0";Od.$inject=["$locale"];Qd.$inject=["$locale"];var ag={yyyy:da("FullYear",4,0,!1,!0),yy:da("FullYear",2,0,!0,!0),y:da("FullYear",1,0,!1,!0),MMMM:sb("Month"),MMM:sb("Month",!0),MM:da("Month",2,1),M:da("Month",1,1),LLLL:sb("Month",!1,!0),dd:da("Date",2),d:da("Date",1),HH:da("Hours",2),H:da("Hours",1),hh:da("Hours",2,-12),h:da("Hours",
1,-12),mm:da("Minutes",2),m:da("Minutes",1),ss:da("Seconds",2),s:da("Seconds",1),sss:da("Milliseconds",3),EEEE:sb("Day"),EEE:sb("Day",!0),a:function(a,b){return 12>a.getHours()?b.AMPMS[0]:b.AMPMS[1]},Z:function(a,b,d){a=-1*d;return(0<=a?"+":"")+(Sb(Math[0<a?"floor":"ceil"](a/60),2)+Sb(Math.abs(a%60),2))},ww:Wd(2),w:Wd(1),G:Kc,GG:Kc,GGG:Kc,GGGG:function(a,b){return 0>=a.getFullYear()?b.ERANAMES[0]:b.ERANAMES[1]}},$f=/((?:[^yMLdHhmsaZEwG']+)|(?:'(?:[^']|'')*')|(?:E+|y+|M+|L+|d+|H+|h+|m+|s+|a|Z|G+|w+))([\s\S]*)/,
Zf=/^-?\d+$/;Pd.$inject=["$locale"];var Uf=la(N),Vf=la(Ob);Rd.$inject=["$parse"];var Cg=la({restrict:"E",compile:function(a,b){if(!b.href&&!b.xlinkHref)return function(a,b){if("a"===b[0].nodeName.toLowerCase()){var c="[object SVGAnimatedString]"===ma.call(b.prop("href"))?"xlink:href":"href";b.on("click",function(a){b.attr(c)||a.preventDefault()})}}}}),Zb={};p(Kb,function(a,b){function d(a,d,e){a.$watch(e[c],function(a){e.$set(b,!!a)})}if("multiple"!==a){var c=Da("ng-"+b),e=d;"checked"===a&&(e=function(a,
b,e){e.ngModel!==e[c]&&d(a,b,e)});Zb[c]=function(){return{restrict:"A",priority:100,link:e}}}});p(rd,function(a,b){Zb[b]=function(){return{priority:100,link:function(a,c,e){if("ngPattern"===b&&"/"===e.ngPattern.charAt(0)&&(c=e.ngPattern.match(ig))){e.$set("ngPattern",new RegExp(c[1],c[2]));return}a.$watch(e[b],function(a){e.$set(b,a)})}}}});p(["src","srcset","href"],function(a){var b=Da("ng-"+a);Zb[b]=function(){return{priority:99,link:function(d,c,e){var g=a,f=a;"href"===a&&"[object SVGAnimatedString]"===
ma.call(c.prop("href"))&&(f="xlinkHref",e.$attr[f]="xlink:href",g=null);e.$observe(b,function(b){b?(e.$set(f,b),La&&g&&c.prop(g,e[f])):"href"===a&&e.$set(f,null)})}}}});var Ub={$addControl:H,$$renameControl:function(a,b){a.$name=b},$removeControl:H,$setValidity:H,$setDirty:H,$setPristine:H,$setSubmitted:H};Tb.$inject=["$element","$attrs","$scope","$animate","$interpolate"];Tb.prototype={$rollbackViewValue:function(){p(this.$$controls,function(a){a.$rollbackViewValue()})},$commitViewValue:function(){p(this.$$controls,
function(a){a.$commitViewValue()})},$addControl:function(a){Ia(a.$name,"input");this.$$controls.push(a);a.$name&&(this[a.$name]=a);a.$$parentForm=this},$$renameControl:function(a,b){var d=a.$name;this[d]===a&&delete this[d];this[b]=a;a.$name=b},$removeControl:function(a){a.$name&&this[a.$name]===a&&delete this[a.$name];p(this.$pending,function(b,d){this.$setValidity(d,null,a)},this);p(this.$error,function(b,d){this.$setValidity(d,null,a)},this);p(this.$$success,function(b,d){this.$setValidity(d,null,
a)},this);eb(this.$$controls,a);a.$$parentForm=Ub},$setDirty:function(){this.$$animate.removeClass(this.$$element,bb);this.$$animate.addClass(this.$$element,$b);this.$dirty=!0;this.$pristine=!1;this.$$parentForm.$setDirty()},$setPristine:function(){this.$$animate.setClass(this.$$element,bb,$b+" ng-submitted");this.$dirty=!1;this.$pristine=!0;this.$submitted=!1;p(this.$$controls,function(a){a.$setPristine()})},$setUntouched:function(){p(this.$$controls,function(a){a.$setUntouched()})},$setSubmitted:function(){this.$$animate.addClass(this.$$element,
"ng-submitted");this.$submitted=!0;this.$$parentForm.$setSubmitted()}};Zd({clazz:Tb,set:function(a,b,d){var c=a[b];c?-1===c.indexOf(d)&&c.push(d):a[b]=[d]},unset:function(a,b,d){var c=a[b];c&&(eb(c,d),0===c.length&&delete a[b])}});var he=function(a){return["$timeout","$parse",function(b,d){function c(a){return""===a?d('this[""]').assign:d(a).assign||H}return{name:"form",restrict:a?"EAC":"E",require:["form","^^?form"],controller:Tb,compile:function(d,g){d.addClass(bb).addClass(tb);var e=g.name?"name":
a&&g.ngForm?"ngForm":!1;return{pre:function(a,d,f,g){var h=g[0];if(!("action"in f)){var k=function(b){a.$apply(function(){h.$commitViewValue();h.$setSubmitted()});b.preventDefault()};d[0].addEventListener("submit",k);d.on("$destroy",function(){b(function(){d[0].removeEventListener("submit",k)},0,!1)})}(g[1]||h.$$parentForm).$addControl(h);var l=e?c(h.$name):H;e&&(l(a,h),f.$observe(e,function(b){h.$name!==b&&(l(a,void 0),h.$$parentForm.$$renameControl(h,b),l=c(h.$name),l(a,h))}));d.on("$destroy",function(){h.$$parentForm.$removeControl(h);
l(a,void 0);P(h,Ub)})}}}}}]},Dg=he(),Eg=he(!0),bg=/^\d{4,}-[01]\d-[0-3]\dT[0-2]\d:[0-5]\d:[0-5]\d\.\d+(?:[+-][0-2]\d:[0-5]\d|Z)$/,Fg=/^[a-z][a-z\d.+-]*:\/*(?:[^:@]+(?::[^@]+)?@)?(?:[^\s:/?#]+|\[[a-f\d:]+])(?::\d+)?(?:\/[^?#]*)?(?:\?[^#]*)?(?:#.*)?$/i,Gg=/^(?=.{1,254}$)(?=.{1,64}@)[-!#$%&'*+/0-9=?A-Z^_`a-z{|}~]+(\.[-!#$%&'*+/0-9=?A-Z^_`a-z{|}~]+)*@[A-Za-z0-9]([A-Za-z0-9-]{0,61}[A-Za-z0-9])?(\.[A-Za-z0-9]([A-Za-z0-9-]{0,61}[A-Za-z0-9])?)*$/,cg=/^\s*(-|\+)?(\d+|(\d*(\.\d*)))([eE][+-]?\d+)?\s*$/,ie=/^(\d{4,})-(\d{2})-(\d{2})$/,
je=/^(\d{4,})-(\d\d)-(\d\d)T(\d\d):(\d\d)(?::(\d\d)(\.\d{1,3})?)?$/,Tc=/^(\d{4,})-W(\d\d)$/,ke=/^(\d{4,})-(\d\d)$/,le=/^(\d\d):(\d\d)(?::(\d\d)(\.\d{1,3})?)?$/,ae=V();p(["date","datetime-local","month","time","week"],function(a){ae[a]=!0});var me={text:function(a,b,d,c,e,g){Ua(a,b,d,c,e,g);Mc(c)},date:ub("date",ie,Vb(ie,["yyyy","MM","dd"]),"yyyy-MM-dd"),"datetime-local":ub("datetimelocal",je,Vb(je,"yyyy MM dd HH mm ss sss".split(" ")),"yyyy-MM-ddTHH:mm:ss.sss"),time:ub("time",le,Vb(le,["HH","mm",
"ss","sss"]),"HH:mm:ss.sss"),week:ub("week",Tc,function(a,b){if(ia(a))return a;if(I(a)){Tc.lastIndex=0;var d=Tc.exec(a);if(d){a=+d[1];var c=+d[2],e=d=0,g=0,f=0,h=Vd(a);c=7*(c-1);b&&(d=b.getHours(),e=b.getMinutes(),g=b.getSeconds(),f=b.getMilliseconds());return new Date(a,0,h.getDate()+c,d,e,g,f)}}return NaN},"yyyy-Www"),month:ub("month",ke,Vb(ke,["yyyy","MM"]),"yyyy-MM"),number:function(a,b,d,c,e,g){Nc(a,b,d,c);be(c);Ua(a,b,d,c,e,g);var f,h;if(v(d.min)||d.ngMin)c.$validators.min=function(a){return c.$isEmpty(a)||
z(f)||a>=f},d.$observe("min",function(a){f=Va(a);c.$validate()});if(v(d.max)||d.ngMax)c.$validators.max=function(a){return c.$isEmpty(a)||z(h)||a<=h},d.$observe("max",function(a){h=Va(a);c.$validate()});if(v(d.step)||d.ngStep){var k;c.$validators.step=function(a,b){return c.$isEmpty(b)||z(k)||ce(b,f||0,k)};d.$observe("step",function(a){k=Va(a);c.$validate()})}},url:function(a,b,d,c,e,g){Ua(a,b,d,c,e,g);Mc(c);c.$$parserName="url";c.$validators.url=function(a,b){a=a||b;return c.$isEmpty(a)||Fg.test(a)}},
email:function(a,b,d,c,e,g){Ua(a,b,d,c,e,g);Mc(c);c.$$parserName="email";c.$validators.email=function(a,b){a=a||b;return c.$isEmpty(a)||Gg.test(a)}},radio:function(a,b,d,c){var e=!d.ngTrim||"false"!==T(d.ngTrim);z(d.name)&&b.attr("name",++wb);b.on("click",function(a){if(b[0].checked){var f=d.value;e&&(f=T(f));c.$setViewValue(f,a&&a.type)}});c.$render=function(){var a=d.value;e&&(a=T(a));b[0].checked=a===c.$viewValue};d.$observe("value",c.$render)},range:function(a,b,d,c,e,g){function f(a,c){b.attr(a,
d[a]);d.$observe(a,c)}function h(a){n=Va(a);aa(c.$modelValue)||(m?(a=b.val(),n>a&&(a=n,b.val(a)),c.$setViewValue(a)):c.$validate())}function k(a){t=Va(a);aa(c.$modelValue)||(m?(a=b.val(),t<a&&(b.val(t),a=t<n?n:t),c.$setViewValue(a)):c.$validate())}function l(a){p=Va(a);aa(c.$modelValue)||(m&&c.$viewValue!==b.val()?c.$setViewValue(b.val()):c.$validate())}Nc(a,b,d,c);be(c);Ua(a,b,d,c,e,g);var m=c.$$hasNativeValidators&&"range"===b[0].type,n=m?0:void 0,t=m?100:void 0,p=m?1:void 0,q=b[0].validity;a=v(d.min);
e=v(d.max);g=v(d.step);var u=c.$render;c.$render=m&&v(q.rangeUnderflow)&&v(q.rangeOverflow)?function(){u();c.$setViewValue(b.val())}:u;a&&(c.$validators.min=m?function(){return!0}:function(a,b){return c.$isEmpty(b)||z(n)||b>=n},f("min",h));e&&(c.$validators.max=m?function(){return!0}:function(a,b){return c.$isEmpty(b)||z(t)||b<=t},f("max",k));g&&(c.$validators.step=m?function(){return!q.stepMismatch}:function(a,b){return c.$isEmpty(b)||z(p)||ce(b,n||0,p)},f("step",l))},checkbox:function(a,b,d,c,e,
g,f,h){var k=de(h,a,"ngTrueValue",d.ngTrueValue,!0),l=de(h,a,"ngFalseValue",d.ngFalseValue,!1);b.on("click",function(a){c.$setViewValue(b[0].checked,a&&a.type)});c.$render=function(){b[0].checked=c.$viewValue};c.$isEmpty=function(a){return!1===a};c.$formatters.push(function(a){return va(a,k)});c.$parsers.push(function(a){return a?k:l})},hidden:H,button:H,submit:H,reset:H,file:H},ne=["$browser","$sniffer","$filter","$parse",function(a,b,d,c){return{restrict:"E",require:["?ngModel"],link:{pre:function(e,
g,f,h){h[0]&&(me[N(f.type)]||me.text)(e,g,f,h[0],b,a,d,c)}}}}],Hg=function(){var a={configurable:!0,enumerable:!1,get:function(){return this.getAttribute("value")||""},set:function(a){this.setAttribute("value",a)}};return{restrict:"E",priority:200,compile:function(b,d){if("hidden"===N(d.type))return{pre:function(b,d,g,f){b=d[0];b.parentNode&&b.parentNode.insertBefore(b,b.nextSibling);Object.defineProperty&&Object.defineProperty(b,"value",a)}}}}},Ig=/^(true|false|\d+)$/,Jg=function(){function a(a,
d,c){var b=v(c)?c:9===La?"":null;a.prop("value",b);d.$set("value",c)}return{restrict:"A",priority:100,compile:function(b,d){return Ig.test(d.ngValue)?function(b,d,g){b=b.$eval(g.ngValue);a(d,g,b)}:function(b,d,g){b.$watch(g.ngValue,function(b){a(d,g,b)})}}}},Kg=["$compile",function(a){return{restrict:"AC",compile:function(b){a.$$addBindingClass(b);return function(b,c,e){a.$$addBindingInfo(c,e.ngBind);c=c[0];b.$watch(e.ngBind,function(a){c.textContent=mc(a)})}}}}],Lg=["$interpolate","$compile",function(a,
b){return{compile:function(d){b.$$addBindingClass(d);return function(c,d,g){c=a(d.attr(g.$attr.ngBindTemplate));b.$$addBindingInfo(d,c.expressions);d=d[0];g.$observe("ngBindTemplate",function(a){d.textContent=z(a)?"":a})}}}}],Mg=["$sce","$parse","$compile",function(a,b,d){return{restrict:"A",compile:function(c,e){var g=b(e.ngBindHtml),f=b(e.ngBindHtml,function(b){return a.valueOf(b)});d.$$addBindingClass(c);return function(b,c,e){d.$$addBindingInfo(c,e.ngBindHtml);b.$watch(f,function(){var d=g(b);
c.html(a.getTrustedHtml(d)||"")})}}}}],Ng=la({restrict:"A",require:"ngModel",link:function(a,b,d,c){c.$viewChangeListeners.push(function(){a.$eval(d.ngChange)})}}),Og=Pc("",!0),Pg=Pc("Odd",0),Qg=Pc("Even",1),Rg=Pa({compile:function(a,b){b.$set("ngCloak",void 0);a.removeClass("ng-cloak")}}),Sg=[function(){return{restrict:"A",scope:!0,controller:"@",priority:500}}],oe={},Tg={blur:!0,focus:!0};p("click dblclick mousedown mouseup mouseover mouseout mousemove mouseenter mouseleave keydown keyup keypress submit focus blur copy cut paste".split(" "),
function(a){var b=Da("ng-"+a);oe[b]=["$parse","$rootScope",function(d,c){return{restrict:"A",compile:function(e,g){var f=d(g[b]);return function(b,d){d.on(a,function(d){var e=function(){f(b,{$event:d})};Tg[a]&&c.$$phase?b.$evalAsync(e):b.$apply(e)})}}}}]});var Ug=["$animate","$compile",function(a,b){return{multiElement:!0,transclude:"element",priority:600,terminal:!0,restrict:"A",$$tlb:!0,link:function(d,c,e,g,f){var h,k,l;d.$watch(e.ngIf,function(d){d?k||f(function(d,f){k=f;d[d.length++]=b.$$createComment("end ngIf",
e.ngIf);h={clone:d};a.enter(d,c.parent(),c)}):(l&&(l.remove(),l=null),k&&(k.$destroy(),k=null),h&&(l=Ab(h.clone),a.leave(l).done(function(a){!1!==a&&(l=null)}),h=null))})}}}],Vg=["$templateRequest","$anchorScroll","$animate",function(a,b,d){return{restrict:"ECA",priority:400,terminal:!0,transclude:"element",controller:ja.noop,compile:function(c,e){var g=e.ngInclude||e.src,f=e.onload||"",h=e.autoscroll;return function(c,e,m,n,t){var k=0,l,p,q,r=function(){p&&(p.remove(),p=null);l&&(l.$destroy(),l=
null);q&&(d.leave(q).done(function(a){!1!==a&&(p=null)}),p=q,q=null)};c.$watch(g,function(g){var m=function(a){!1===a||!v(h)||h&&!c.$eval(h)||b()},p=++k;g?(a(g,!0).then(function(a){if(!c.$$destroyed&&p===k){var b=c.$new();n.template=a;a=t(b,function(a){r();d.enter(a,null,e).done(m)});l=b;q=a;l.$emit("$includeContentLoaded",g);c.$eval(f)}},function(){c.$$destroyed||p!==k||(r(),c.$emit("$includeContentError",g))}),c.$emit("$includeContentRequested",g)):(r(),n.template=null)})}}}}],Wg=["$compile",function(a){return{restrict:"ECA",
priority:-400,require:"ngInclude",link:function(b,d,c,e){ma.call(d[0]).match(/SVG/)?(d.empty(),a(ed(e.template,D.document).childNodes)(b,function(a){d.append(a)},{futureParentElement:d})):(d.html(e.template),a(d.contents())(b))}}}],Xg=Pa({priority:450,compile:function(){return{pre:function(a,b,d){a.$eval(d.ngInit)}}}}),Yg=function(){return{restrict:"A",priority:100,require:"ngModel",link:function(a,b,d,c){var e=d.ngList||", ",g="false"!==d.ngTrim,f=g?T(e):e;c.$parsers.push(function(a){if(!z(a)){var b=
[];a&&p(a.split(f),function(a){a&&b.push(g?T(a):a)});return b}});c.$formatters.push(function(a){if(J(a))return a.join(e)});c.$isEmpty=function(a){return!a||!a.length}}}},tb="ng-valid",Yd="ng-invalid",bb="ng-pristine",$b="ng-dirty",vb=K("ngModel");Wb.$inject="$scope $exceptionHandler $attrs $element $parse $animate $timeout $q $interpolate".split(" ");Wb.prototype={$$initGetterSetters:function(){if(this.$options.getOption("getterSetter")){var a=this.$$parse(this.$$attr.ngModel+"()"),b=this.$$parse(this.$$attr.ngModel+
"($$$p)");this.$$ngModelGet=function(b){var c=this.$$parsedNgModel(b);A(c)&&(c=a(b));return c};this.$$ngModelSet=function(a,c){A(this.$$parsedNgModel(a))?b(a,{$$$p:c}):this.$$parsedNgModelAssign(a,c)}}else if(!this.$$parsedNgModel.assign)throw vb("nonassign",this.$$attr.ngModel,Aa(this.$$element));},$render:H,$isEmpty:function(a){return z(a)||""===a||null===a||a!==a},$$updateEmptyClasses:function(a){this.$isEmpty(a)?(this.$$animate.removeClass(this.$$element,"ng-not-empty"),this.$$animate.addClass(this.$$element,
"ng-empty")):(this.$$animate.removeClass(this.$$element,"ng-empty"),this.$$animate.addClass(this.$$element,"ng-not-empty"))},$setPristine:function(){this.$dirty=!1;this.$pristine=!0;this.$$animate.removeClass(this.$$element,$b);this.$$animate.addClass(this.$$element,bb)},$setDirty:function(){this.$dirty=!0;this.$pristine=!1;this.$$animate.removeClass(this.$$element,bb);this.$$animate.addClass(this.$$element,$b);this.$$parentForm.$setDirty()},$setUntouched:function(){this.$touched=!1;this.$untouched=
!0;this.$$animate.setClass(this.$$element,"ng-untouched","ng-touched")},$setTouched:function(){this.$touched=!0;this.$untouched=!1;this.$$animate.setClass(this.$$element,"ng-touched","ng-untouched")},$rollbackViewValue:function(){this.$$timeout.cancel(this.$$pendingDebounce);this.$viewValue=this.$$lastCommittedViewValue;this.$render()},$validate:function(){if(!aa(this.$modelValue)){var a=this.$$lastCommittedViewValue,b=this.$$rawModelValue,d=this.$valid,c=this.$modelValue,e=this.$options.getOption("allowInvalid"),
g=this;this.$$runValidators(b,a,function(a){e||d===a||(g.$modelValue=a?b:void 0,g.$modelValue!==c&&g.$$writeModelToScope())})}},$$runValidators:function(a,b,d){function c(a,b){g===f.$$currentValidationRunId&&f.$setValidity(a,b)}function e(a){g===f.$$currentValidationRunId&&d(a)}this.$$currentValidationRunId++;var g=this.$$currentValidationRunId,f=this;(function(){var a=f.$$parserName||"parse";if(z(f.$$parserValid))c(a,null);else return f.$$parserValid||(p(f.$validators,function(a,b){c(b,null)}),p(f.$asyncValidators,
function(a,b){c(b,null)})),c(a,f.$$parserValid),f.$$parserValid;return!0})()?function(){var d=!0;p(f.$validators,function(e,g){e=!!e(a,b);d=d&&e;c(g,e)});return d?!0:(p(f.$asyncValidators,function(a,b){c(b,null)}),!1)}()?function(){var d=[],g=!0;p(f.$asyncValidators,function(e,f){e=e(a,b);if(!e||!A(e.then))throw vb("nopromise",e);c(f,void 0);d.push(e.then(function(){c(f,!0)},function(){g=!1;c(f,!1)}))});d.length?f.$$q.all(d).then(function(){e(g)},H):e(!0)}():e(!1):e(!1)},$commitViewValue:function(){var a=
this.$viewValue;this.$$timeout.cancel(this.$$pendingDebounce);if(this.$$lastCommittedViewValue!==a||""===a&&this.$$hasNativeValidators)this.$$updateEmptyClasses(a),this.$$lastCommittedViewValue=a,this.$pristine&&this.$setDirty(),this.$$parseAndValidate()},$$parseAndValidate:function(){var a=this.$$lastCommittedViewValue,b=this;if(this.$$parserValid=z(a)?void 0:!0)for(var d=0;d<this.$parsers.length;d++)if(a=this.$parsers[d](a),z(a)){this.$$parserValid=!1;break}aa(this.$modelValue)&&(this.$modelValue=
this.$$ngModelGet(this.$$scope));var c=this.$modelValue,e=this.$options.getOption("allowInvalid");this.$$rawModelValue=a;e&&(this.$modelValue=a,b.$modelValue!==c&&b.$$writeModelToScope());this.$$runValidators(a,this.$$lastCommittedViewValue,function(d){e||(b.$modelValue=d?a:void 0,b.$modelValue!==c&&b.$$writeModelToScope())})},$$writeModelToScope:function(){this.$$ngModelSet(this.$$scope,this.$modelValue);p(this.$viewChangeListeners,function(a){try{a()}catch(b){this.$$exceptionHandler(b)}},this)},
$setViewValue:function(a,b){this.$viewValue=a;this.$options.getOption("updateOnDefault")&&this.$$debounceViewValueCommit(b)},$$debounceViewValueCommit:function(a){var b=this.$options.getOption("debounce");ca(b[a])?b=b[a]:ca(b["default"])&&(b=b["default"]);this.$$timeout.cancel(this.$$pendingDebounce);var d=this;0<b?this.$$pendingDebounce=this.$$timeout(function(){d.$commitViewValue()},b):this.$$scope.$root.$$phase?this.$commitViewValue():this.$$scope.$apply(function(){d.$commitViewValue()})},$overrideModelOptions:function(a){this.$options=
this.$options.createChild(a);this.$$setUpdateOnEvents()},$processModelValue:function(){var a=this.$$format();this.$viewValue!==a&&(this.$$updateEmptyClasses(a),this.$viewValue=this.$$lastCommittedViewValue=a,this.$render(),this.$$runValidators(this.$modelValue,this.$viewValue,H))},$$format:function(){for(var a=this.$formatters,b=a.length,d=this.$modelValue;b--;)d=a[b](d);return d},$$setModelValue:function(a){this.$modelValue=this.$$rawModelValue=a;this.$$parserValid=void 0;this.$processModelValue()},
$$setUpdateOnEvents:function(){this.$$updateEvents&&this.$$element.off(this.$$updateEvents,this.$$updateEventHandler);if(this.$$updateEvents=this.$options.getOption("updateOn"))this.$$element.on(this.$$updateEvents,this.$$updateEventHandler)},$$updateEventHandler:function(a){this.$$debounceViewValueCommit(a&&a.type)}};Zd({clazz:Wb,set:function(a,b){a[b]=!0},unset:function(a,b){delete a[b]}});var Zg=["$rootScope",function(a){return{restrict:"A",require:["ngModel","^?form","^?ngModelOptions"],controller:Wb,
priority:1,compile:function(b){b.addClass(bb).addClass("ng-untouched").addClass(tb);return{pre:function(a,b,e,g){var c=g[0];b=g[1]||c.$$parentForm;if(g=g[2])c.$options=g.$options;c.$$initGetterSetters();b.$addControl(c);e.$observe("name",function(a){c.$name!==a&&c.$$parentForm.$$renameControl(c,a)});a.$on("$destroy",function(){c.$$parentForm.$removeControl(c)})},post:function(b,c,e,g){function d(){h.$setTouched()}var h=g[0];h.$$setUpdateOnEvents();c.on("blur",function(){h.$touched||(a.$$phase?b.$evalAsync(d):
b.$apply(d))})}}}}}],$g=/(\s+|^)default(\s+|$)/;Rc.prototype={getOption:function(a){return this.$$options[a]},createChild:function(a){var b=!1;a=P({},a);p(a,function(d,c){"$inherit"===d?"*"===c?b=!0:(a[c]=this.$$options[c],"updateOn"===c&&(a.updateOnDefault=this.$$options.updateOnDefault)):"updateOn"===c&&(a.updateOnDefault=!1,a[c]=T(d.replace($g,function(){a.updateOnDefault=!0;return" "})))},this);b&&(delete a["*"],ee(a,this.$$options));ee(a,Qc.$$options);return new Rc(a)}};var Qc=new Rc({updateOn:"",
updateOnDefault:!0,debounce:0,getterSetter:!1,allowInvalid:!1,timezone:null});var ah=function(){function a(a,d){this.$$attrs=a;this.$$scope=d}a.$inject=["$attrs","$scope"];a.prototype={$onInit:function(){var a=this.parentCtrl?this.parentCtrl.$options:Qc,d=this.$$scope.$eval(this.$$attrs.ngModelOptions);this.$options=a.createChild(d)}};return{restrict:"A",priority:10,require:{parentCtrl:"?^^ngModelOptions"},bindToController:!0,controller:a}},bh=Pa({terminal:!0,priority:1E3}),ch=K("ngOptions"),dh=/^\s*([\s\S]+?)(?:\s+as\s+([\s\S]+?))?(?:\s+group\s+by\s+([\s\S]+?))?(?:\s+disable\s+when\s+([\s\S]+?))?\s+for\s+(?:([$\w][$\w]*)|(?:\(\s*([$\w][$\w]*)\s*,\s*([$\w][$\w]*)\s*\)))\s+in\s+([\s\S]+?)(?:\s+track\s+by\s+([\s\S]+?))?$/,
eh=["$compile","$document","$parse",function(a,b,d){function c(a,b,c){function e(a,b,c,d,e){this.selectValue=a;this.viewValue=b;this.label=c;this.group=d;this.disabled=e}function g(a){if(!k&&pa(a))var b=a;else{b=[];for(var c in a)a.hasOwnProperty(c)&&"$"!==c.charAt(0)&&b.push(c)}return b}var f=a.match(dh);if(!f)throw ch("iexp",a,Aa(b));var h=f[5]||f[7],k=f[6];a=/ as /.test(f[0])&&f[1];var p=f[9];b=d(f[2]?f[1]:h);var q=a&&d(a)||b,v=p&&d(p),r=p?function(a,b){return v(c,b)}:function(a){return Ja(a)},
x=function(a,b){return r(a,D(a,b))},z=d(f[2]||f[1]),y=d(f[3]||""),C=d(f[4]||""),w=d(f[8]),A={},D=k?function(a,b){A[k]=b;A[h]=a;return A}:function(a){A[h]=a;return A};return{trackBy:p,getTrackByValue:x,getWatchables:d(w,function(a){var b=[];a=a||[];for(var d=g(a),e=d.length,h=0;h<e;h++){var k=a===d?h:d[h],l=a[k];k=D(l,k);l=r(l,k);b.push(l);if(f[2]||f[1])l=z(c,k),b.push(l);f[4]&&(k=C(c,k),b.push(k))}return b}),getOptions:function(){for(var a=[],b={},d=w(c)||[],f=g(d),h=f.length,k=0;k<h;k++){var l=d===
f?k:f[k],m=D(d[l],l),n=q(c,m);l=r(n,m);var t=z(c,m),u=y(c,m);m=C(c,m);n=new e(l,n,t,u,m);a.push(n);b[l]=n}return{items:a,selectValueMap:b,getOptionFromViewValue:function(a){return b[x(a)]},getViewValueFromOption:function(a){return p?Ga(a.viewValue):a.viewValue}}}}}var e=D.document.createElement("option"),g=D.document.createElement("optgroup");return{restrict:"A",terminal:!0,require:["select","ngModel"],link:{pre:function(a,b,c,d){d[0].registerOption=H},post:function(d,h,k,l){function f(a){var b=(a=
r.getOptionFromViewValue(a))&&a.element;b&&!b.selected&&(b.selected=!0);return a}function n(a,b){a.element=b;b.disabled=a.disabled;a.label!==b.label&&(b.label=a.label,b.textContent=a.label);b.value=a.selectValue}var t=l[0],q=l[1],B=k.multiple;l=0;for(var u=h.children(),F=u.length;l<F;l++)if(""===u[l].value){t.hasEmptyOption=!0;t.emptyOption=u.eq(l);break}h.empty();l=!!t.emptyOption;x(e.cloneNode(!1)).val("?");var r,z=c(k.ngOptions,h,d),A=b[0].createDocumentFragment();t.generateUnknownOptionValue=
function(a){return"?"};B?(t.writeValue=function(a){if(r){var b=a&&a.map(f)||[];r.items.forEach(function(a){a.element.selected&&-1===Array.prototype.indexOf.call(b,a)&&(a.element.selected=!1)})}},t.readValue=function(){var a=h.val()||[],b=[];p(a,function(a){(a=r.selectValueMap[a])&&!a.disabled&&b.push(r.getViewValueFromOption(a))});return b},z.trackBy&&d.$watchCollection(function(){if(J(q.$viewValue))return q.$viewValue.map(function(a){return z.getTrackByValue(a)})},function(){q.$render()})):(t.writeValue=
function(a){if(r){var b=h[0].options[h[0].selectedIndex],c=r.getOptionFromViewValue(a);b&&b.removeAttribute("selected");c?(h[0].value!==c.selectValue&&(t.removeUnknownOption(),h[0].value=c.selectValue,c.element.selected=!0),c.element.setAttribute("selected","selected")):t.selectUnknownOrEmptyOption(a)}},t.readValue=function(){var a=r.selectValueMap[h.val()];return a&&!a.disabled?(t.unselectEmptyOption(),t.removeUnknownOption(),r.getViewValueFromOption(a)):null},z.trackBy&&d.$watch(function(){return z.getTrackByValue(q.$viewValue)},
function(){q.$render()}));l&&(a(t.emptyOption)(d),h.prepend(t.emptyOption),8===t.emptyOption[0].nodeType?(t.hasEmptyOption=!1,t.registerOption=function(a,b){""===b.val()&&(t.hasEmptyOption=!0,t.emptyOption=b,t.emptyOption.removeClass("ng-scope"),q.$render(),b.on("$destroy",function(){var a=t.$isEmptyOptionSelected();t.hasEmptyOption=!1;t.emptyOption=void 0;a&&q.$render()}))}):t.emptyOption.removeClass("ng-scope"));d.$watchCollection(z.getWatchables,function(){var a=r&&t.readValue();if(r)for(var b=
r.items.length-1;0<=b;b--){var c=r.items[b];v(c.group)?Jb(c.element.parentNode):Jb(c.element)}r=z.getOptions();var d={};r.items.forEach(function(a){if(v(a.group)){var b=d[a.group];b||(b=g.cloneNode(!1),A.appendChild(b),b.label=null===a.group?"null":a.group,d[a.group]=b);var c=e.cloneNode(!1);b.appendChild(c);n(a,c)}else b=e.cloneNode(!1),A.appendChild(b),n(a,b)});h[0].appendChild(A);q.$render();q.$isEmpty(a)||(b=t.readValue(),(z.trackBy||B?va(a,b):a===b)||(q.$setViewValue(b),q.$render()))})}}}}],
fh=["$locale","$interpolate","$log",function(a,b,d){var c=/{}/g,e=/^when(Minus)?(.+)$/;return{link:function(g,f,h){function k(a){f.text(a||"")}var l=h.count,m=h.$attr.when&&f.attr(h.$attr.when),n=h.offset||0,t=g.$eval(m)||{},q={},v=b.startSymbol(),u=b.endSymbol(),x=v+l+"-"+n+u,r=ja.noop,A;p(h,function(a,b){if(a=e.exec(b))a=(a[1]?"-":"")+N(a[2]),t[a]=f.attr(h.$attr[b])});p(t,function(a,d){q[d]=b(a.replace(c,x))});g.$watch(l,function(b){var c=parseFloat(b),e=aa(c);e||c in t||(c=a.pluralCat(c-n));c===
A||e&&aa(A)||(r(),e=q[c],z(e)?(null!=b&&d.debug("ngPluralize: no rule defined for '"+c+"' in "+m),r=H,k()):r=g.$watch(e,k),A=c)})}}}],gh=["$parse","$animate","$compile",function(a,b,d){var c=K("ngRepeat"),e=function(a,b,c,d,e,m,n){a[c]=d;e&&(a[e]=m);a.$index=b;a.$first=0===b;a.$last=b===n-1;a.$middle=!(a.$first||a.$last);a.$odd=!(a.$even=0===(b&1))};return{restrict:"A",multiElement:!0,transclude:"element",priority:1E3,terminal:!0,$$tlb:!0,compile:function(g,f){var h=f.ngRepeat,k=d.$$createComment("end ngRepeat",
h);g=h.match(/^\s*([\s\S]+?)\s+in\s+([\s\S]+?)(?:\s+as\s+([\s\S]+?))?(?:\s+track\s+by\s+([\s\S]+?))?\s*$/);if(!g)throw c("iexp",h);f=g[1];var l=g[2],m=g[3],n=g[4];g=f.match(/^(?:(\s*[$\w]+)|\(\s*([$\w]+)\s*,\s*([$\w]+)\s*\))$/);if(!g)throw c("iidexp",f);var t=g[3]||g[1],q=g[2];if(m&&(!/^[$a-zA-Z_][$a-zA-Z0-9_]*$/.test(m)||/^(null|undefined|this|\$index|\$first|\$middle|\$last|\$even|\$odd|\$parent|\$root|\$id)$/.test(m)))throw c("badident",m);var v,u={$id:Ja};if(n)var x=a(n);else{var r=function(a,
b){return Ja(b)};var z=function(a){return a}}return function(a,d,g,f,n){x&&(v=function(b,c,d){q&&(u[q]=b);u[t]=c;u.$index=d;return x(a,u)});var w=V();a.$watchCollection(l,function(g){var f,l=d[0],u=V();m&&(a[m]=g);if(pa(g)){var x=g;var F=v||r}else for(E in F=v||z,x=[],g)ua.call(g,E)&&"$"!==E.charAt(0)&&x.push(E);var B=x.length;var E=Array(B);for(f=0;f<B;f++){var A=g===x?f:x[f];var C=g[A];var y=F(A,C,f);if(w[y]){var D=w[y];delete w[y];u[y]=D;E[f]=D}else{if(u[y])throw p(E,function(a){a&&a.scope&&(w[a.id]=
a)}),c("dupes",h,y,C);E[f]={id:y,scope:void 0,clone:void 0};u[y]=!0}}for(G in w){D=w[G];y=Ab(D.clone);b.leave(y);if(y[0].parentNode)for(f=0,F=y.length;f<F;f++)y[f].$$NG_REMOVED=!0;D.scope.$destroy()}for(f=0;f<B;f++)if(A=g===x?f:x[f],C=g[A],D=E[f],D.scope){var G=l;do G=G.nextSibling;while(G&&G.$$NG_REMOVED);D.clone[0]!==G&&b.move(Ab(D.clone),null,l);l=D.clone[D.clone.length-1];e(D.scope,f,t,C,q,A,B)}else n(function(a,c){D.scope=c;c=k.cloneNode(!1);a[a.length++]=c;b.enter(a,null,l);l=c;D.clone=a;u[D.id]=
D;e(D.scope,f,t,C,q,A,B)});w=u})}}}}],hh=["$animate",function(a){return{restrict:"A",multiElement:!0,link:function(b,d,c){b.$watch(c.ngShow,function(b){a[b?"removeClass":"addClass"](d,"ng-hide",{tempClasses:"ng-hide-animate"})})}}}],ih=["$animate",function(a){return{restrict:"A",multiElement:!0,link:function(b,d,c){b.$watch(c.ngHide,function(b){a[b?"addClass":"removeClass"](d,"ng-hide",{tempClasses:"ng-hide-animate"})})}}}],jh=Pa(function(a,b,d){a.$watch(d.ngStyle,function(a,d){d&&a!==d&&p(d,function(a,
c){b.css(c,"")});a&&b.css(a)},!0)}),kh=["$animate","$compile",function(a,b){return{require:"ngSwitch",controller:["$scope",function(){this.cases={}}],link:function(d,c,e,g){var f=[],h=[],k=[],l=[],m=function(a,b){return function(c){!1!==c&&a.splice(b,1)}};d.$watch(e.ngSwitch||e.on,function(c){for(var d,e;k.length;)a.cancel(k.pop());d=0;for(e=l.length;d<e;++d){var n=Ab(h[d].clone);l[d].$destroy();(k[d]=a.leave(n)).done(m(k,d))}h.length=0;l.length=0;(f=g.cases["!"+c]||g.cases["?"])&&p(f,function(c){c.transclude(function(d,
e){l.push(e);e=c.element;d[d.length++]=b.$$createComment("end ngSwitchWhen");h.push({clone:d});a.enter(d,e.parent(),e)})})})}}}],lh=Pa({transclude:"element",priority:1200,require:"^ngSwitch",multiElement:!0,link:function(a,b,d,c,e){a=d.ngSwitchWhen.split(d.ngSwitchWhenSeparator).sort().filter(function(a,b,c){return c[b-1]!==a});p(a,function(a){c.cases["!"+a]=c.cases["!"+a]||[];c.cases["!"+a].push({transclude:e,element:b})})}}),mh=Pa({transclude:"element",priority:1200,require:"^ngSwitch",multiElement:!0,
link:function(a,b,d,c,e){c.cases["?"]=c.cases["?"]||[];c.cases["?"].push({transclude:e,element:b})}}),nh=K("ngTransclude"),oh=["$compile",function(a){return{restrict:"EAC",compile:function(b){var d=a(b.contents());b.empty();return function(a,b,g,f,h){function c(){d(a,function(a){b.append(a)})}if(!h)throw nh("orphan",Aa(b));g.ngTransclude===g.$attr.ngTransclude&&(g.ngTransclude="");g=g.ngTransclude||g.ngTranscludeSlot;h(function(a,d){var e;if(e=a.length)a:{e=0;for(var f=a.length;e<f;e++){var g=a[e];
if(g.nodeType!==Sa||g.nodeValue.trim()){e=!0;break a}}e=void 0}e?b.append(a):(c(),d.$destroy())},null,g);g&&!h.isSlotFilled(g)&&c()}}}}],ph=["$templateCache",function(a){return{restrict:"E",terminal:!0,compile:function(b,d){"text/ng-template"===d.type&&a.put(d.id,b[0].text)}}}],qh={$setViewValue:H,$render:H},rh=["$element","$scope",function(a,b){function d(){f||(f=!0,b.$$postDigest(function(){f=!1;e.ngModelCtrl.$render()}))}function c(a){h||(h=!0,b.$$postDigest(function(){b.$$destroyed||(h=!1,e.ngModelCtrl.$setViewValue(e.readValue()),
a&&e.ngModelCtrl.$render())}))}var e=this,g=new Lb;e.selectValueMap={};e.ngModelCtrl=qh;e.multiple=!1;e.unknownOption=x(D.document.createElement("option"));e.hasEmptyOption=!1;e.emptyOption=void 0;e.renderUnknownOption=function(b){b=e.generateUnknownOptionValue(b);e.unknownOption.val(b);a.prepend(e.unknownOption);Qa(e.unknownOption,!0);a.val(b)};e.updateUnknownOption=function(b){b=e.generateUnknownOptionValue(b);e.unknownOption.val(b);Qa(e.unknownOption,!0);a.val(b)};e.generateUnknownOptionValue=
function(a){return"? "+Ja(a)+" ?"};e.removeUnknownOption=function(){e.unknownOption.parent()&&e.unknownOption.remove()};e.selectEmptyOption=function(){e.emptyOption&&(a.val(""),Qa(e.emptyOption,!0))};e.unselectEmptyOption=function(){e.hasEmptyOption&&Qa(e.emptyOption,!1)};b.$on("$destroy",function(){e.renderUnknownOption=H});e.readValue=function(){var b=a.val();b=b in e.selectValueMap?e.selectValueMap[b]:b;return e.hasOption(b)?b:null};e.writeValue=function(b){var c=a[0].options[a[0].selectedIndex];
c&&Qa(x(c),!1);e.hasOption(b)?(e.removeUnknownOption(),c=Ja(b),a.val(c in e.selectValueMap?c:b),Qa(x(a[0].options[a[0].selectedIndex]),!0)):e.selectUnknownOrEmptyOption(b)};e.addOption=function(a,b){8!==b[0].nodeType&&(Ia(a,'"option value"'),""===a&&(e.hasEmptyOption=!0,e.emptyOption=b),b=g.get(a)||0,g.set(a,b+1),d())};e.removeOption=function(a){var b=g.get(a);b&&(1===b?(g.delete(a),""===a&&(e.hasEmptyOption=!1,e.emptyOption=void 0)):g.set(a,b-1))};e.hasOption=function(a){return!!g.get(a)};e.$hasEmptyOption=
function(){return e.hasEmptyOption};e.$isUnknownOptionSelected=function(){return a[0].options[0]===e.unknownOption[0]};e.$isEmptyOptionSelected=function(){return e.hasEmptyOption&&a[0].options[a[0].selectedIndex]===e.emptyOption[0]};e.selectUnknownOrEmptyOption=function(a){null==a&&e.emptyOption?(e.removeUnknownOption(),e.selectEmptyOption()):e.unknownOption.parent().length?e.updateUnknownOption(a):e.renderUnknownOption(a)};var f=!1,h=!1;e.registerOption=function(a,b,f,g,h){if(f.$attr.ngValue){var k,
l=NaN;f.$observe("value",function(a){var d=b.prop("selected");if(v(l)){e.removeOption(k);delete e.selectValueMap[l];var f=!0}l=Ja(a);k=a;e.selectValueMap[l]=a;e.addOption(a,b);b.attr("value",l);f&&d&&c()})}else g?f.$observe("value",function(a){e.readValue();var d=b.prop("selected");if(v(k)){e.removeOption(k);var f=!0}k=a;e.addOption(a,b);f&&d&&c()}):h?a.$watch(h,function(a,d){f.$set("value",a);var g=b.prop("selected");d!==a&&e.removeOption(d);e.addOption(a,b);d&&g&&c()}):e.addOption(f.value,b);f.$observe("disabled",
function(a){if("true"===a||a&&b.prop("selected"))e.multiple?c(!0):(e.ngModelCtrl.$setViewValue(null),e.ngModelCtrl.$render())});b.on("$destroy",function(){var a=e.readValue(),b=f.value;e.removeOption(b);d();(e.multiple&&a&&-1!==a.indexOf(b)||a===b)&&c(!0)})}}],sh=function(){return{restrict:"E",require:["select","?ngModel"],controller:rh,priority:1,link:{pre:function(a,b,d,c){var e=c[0],g=c[1];if(g){if(e.ngModelCtrl=g,b.on("change",function(){e.removeUnknownOption();a.$apply(function(){g.$setViewValue(e.readValue())})}),
d.multiple){e.multiple=!0;e.readValue=function(){var a=[];p(b.find("option"),function(b){b.selected&&!b.disabled&&(b=b.value,a.push(b in e.selectValueMap?e.selectValueMap[b]:b))});return a};e.writeValue=function(a){p(b.find("option"),function(b){var c=!!a&&(-1!==Array.prototype.indexOf.call(a,b.value)||-1!==Array.prototype.indexOf.call(a,e.selectValueMap[b.value]));c!==b.selected&&Qa(x(b),c)})};var f,h=NaN;a.$watch(function(){h!==g.$viewValue||va(f,g.$viewValue)||(f=ra(g.$viewValue),g.$render());
h=g.$viewValue});g.$isEmpty=function(a){return!a||0===a.length}}}else e.registerOption=H},post:function(a,b,d,c){var e=c[1];if(e){var g=c[0];e.$render=function(){g.writeValue(e.$viewValue)}}}}}},th=["$interpolate",function(a){return{restrict:"E",priority:100,compile:function(b,d){var c;if(!v(d.ngValue))if(v(d.value))var e=a(d.value,!0);else(c=a(b.text(),!0))||d.$set("value",b.text());return function(a,b,d){var f=b.parent();(f=f.data("$selectController")||f.parent().data("$selectController"))&&f.registerOption(a,
b,d,e,c)}}}}],pe=function(){return{restrict:"A",require:"?ngModel",link:function(a,b,d,c){c&&(d.required=!0,c.$validators.required=function(a,b){return!d.required||!c.$isEmpty(b)},d.$observe("required",function(){c.$validate()}))}}},qe=function(){return{restrict:"A",require:"?ngModel",link:function(a,b,d,c){if(c){var e,g=d.ngPattern||d.pattern;d.$observe("pattern",function(a){I(a)&&0<a.length&&(a=new RegExp("^"+a+"$"));if(a&&!a.test)throw K("ngPattern")("noregexp",g,a,Aa(b));e=a||void 0;c.$validate()});
c.$validators.pattern=function(a,b){return c.$isEmpty(b)||z(e)||e.test(b)}}}}},re=function(){return{restrict:"A",require:"?ngModel",link:function(a,b,d,c){if(c){var e=-1;d.$observe("maxlength",function(a){a=parseInt(a,10);e=aa(a)?-1:a;c.$validate()});c.$validators.maxlength=function(a,b){return 0>e||c.$isEmpty(b)||b.length<=e}}}}},se=function(){return{restrict:"A",require:"?ngModel",link:function(a,b,d,c){if(c){var e=0;d.$observe("minlength",function(a){e=parseInt(a,10)||0;c.$validate()});c.$validators.minlength=
function(a,b){return c.$isEmpty(b)||b.length>=e}}}}};if(D.angular.bootstrap)D.console&&console.log("WARNING: Tried to load AngularJS more than once.");else{(function(){if(!fe){var a=Xb();if((Fa=z(a)?D.jQuery:a?D[a]:void 0)&&Fa.fn.on){x=Fa;P(Fa.fn,{scope:Za.scope,isolateScope:Za.isolateScope,controller:Za.controller,injector:Za.injector,inheritedData:Za.inheritedData});var b=Fa.cleanData;Fa.cleanData=function(a){for(var c,d=0,g;null!=(g=a[d]);d++)(c=Fa._data(g,"events"))&&c.$destroy&&Fa(g).triggerHandler("$destroy");
b(a)}}else x=Y;ja.element=x;fe=!0}})();(function(a){P(a,{errorHandlingConfig:te,bootstrap:bd,copy:Ga,extend:P,merge:ve,equals:va,element:x,forEach:p,injector:ib,noop:H,bind:Xa,toJson:gb,fromJson:Zc,identity:yb,isUndefined:z,isDefined:v,isString:I,isFunction:A,isObject:G,isNumber:ca,isElement:dc,isArray:J,version:kg,isDate:ia,lowercase:N,uppercase:Ob,callbacks:{$$counter:0},getTestability:Ee,reloadWithDebugInfo:De,$$minErr:K,$$csp:Ea,$$encodeUriSegment:hb,$$encodeUriQuery:na,$$stringify:mc});uc=Ge(D);
uc("ng",["ngLocale"],["$provide",function(a){a.provider({$$sanitizeUri:Lf});a.provider("$compile",nd).directive({a:Cg,input:ne,textarea:ne,form:Dg,script:ph,select:sh,option:th,ngBind:Kg,ngBindHtml:Mg,ngBindTemplate:Lg,ngClass:Og,ngClassEven:Qg,ngClassOdd:Pg,ngCloak:Rg,ngController:Sg,ngForm:Eg,ngHide:ih,ngIf:Ug,ngInclude:Vg,ngInit:Xg,ngNonBindable:bh,ngPluralize:fh,ngRepeat:gh,ngShow:hh,ngStyle:jh,ngSwitch:kh,ngSwitchWhen:lh,ngSwitchDefault:mh,ngOptions:eh,ngTransclude:oh,ngModel:Zg,ngList:Yg,ngChange:Ng,
pattern:qe,ngPattern:qe,required:pe,ngRequired:pe,minlength:se,ngMinlength:se,maxlength:re,ngMaxlength:re,ngValue:Jg,ngModelOptions:ah}).directive({ngInclude:Wg,input:Hg}).directive(Zb).directive(oe);a.provider({$anchorScroll:We,$animate:tg,$animateCss:wg,$$animateJs:rg,$$animateQueue:sg,$$AnimateRunner:vg,$$animateAsyncRun:ug,$browser:Ze,$cacheFactory:$e,$controller:hf,$document:jf,$$isDocumentHidden:kf,$exceptionHandler:lf,$filter:Nd,$$forceReflow:xg,$interpolate:wf,$interval:xf,$http:rf,$httpParamSerializer:mf,
$httpParamSerializerJQLike:nf,$httpBackend:uf,$xhrFactory:tf,$jsonpCallbacks:yg,$location:Af,$log:Bf,$parse:Gf,$rootScope:Kf,$q:Hf,$$q:If,$sce:fg,$sceDelegate:eg,$sniffer:Mf,$templateCache:af,$templateRequest:gg,$$testability:Nf,$timeout:Of,$window:Pf,$$rAF:Jf,$$jqLite:Re,$$Map:og,$$cookieReader:Qf})}]).info({angularVersion:"1.6.4-local+sha.617b36117"})})(ja);ja.module("ngLocale",[],["$provide",function(a){function b(a){a+="";var b=a.indexOf(".");return-1==b?0:a.length-b-1}a.value("$locale",{DATETIME_FORMATS:{AMPMS:["AM",
"PM"],DAY:"Sunday Monday Tuesday Wednesday Thursday Friday Saturday".split(" "),ERANAMES:["Before Christ","Anno Domini"],ERAS:["BC","AD"],FIRSTDAYOFWEEK:6,MONTH:"January February March April May June July August September October November December".split(" "),SHORTDAY:"Sun Mon Tue Wed Thu Fri Sat".split(" "),SHORTMONTH:"Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec".split(" "),STANDALONEMONTH:"January February March April May June July August September October November December".split(" "),WEEKENDRANGE:[5,
6],fullDate:"EEEE, MMMM d, y",longDate:"MMMM d, y",medium:"MMM d, y h:mm:ss a",mediumDate:"MMM d, y",mediumTime:"h:mm:ss a","short":"M/d/yy h:mm a",shortDate:"M/d/yy",shortTime:"h:mm a"},NUMBER_FORMATS:{CURRENCY_SYM:"$",DECIMAL_SEP:".",GROUP_SEP:",",PATTERNS:[{gSize:3,lgSize:3,maxFrac:3,minFrac:0,minInt:1,negPre:"-",negSuf:"",posPre:"",posSuf:""},{gSize:3,lgSize:3,maxFrac:2,minFrac:2,minInt:1,negPre:"-\u00a4",negSuf:"",posPre:"\u00a4",posSuf:""}]},id:"en-us",localeID:"en_US",pluralCat:function(a,
c){var d=a|0;void 0===c&&(c=Math.min(b(a),3));return 1==d&&0==c?"one":"other"}})}]);var fa=K("$sce"),$a={HTML:"html",CSS:"css",URL:"url",RESOURCE_URL:"resourceUrl",TEMPLATE_URL:"templateUrl",JS:"js"},Sc=/_([a-z])/g,hg=K("$compile");x(function(){Be(D.document,bd)})}})(window);angular.element(document).find("head").append('<style type="text/css">@charset "UTF-8";\n\n[ng\\:cloak], [ng-cloak], [data-ng-cloak], [x-ng-cloak],\n.ng-cloak, .x-ng-cloak,\n.ng-hide:not(.ng-hide-animate) {\n  display: none !important;\n}\n\nng\\:form {\n  display: block;\n}\n\n.ng-animate-shim {\n  visibility:hidden;\n}\n\n.ng-anchor {\n  position:absolute;\n}\n</style>');

//third_party/javascript/angular/v1_6/angular-route.min.js
/*
 AngularJS v1.6.4-local+sha.617b36117
 (c) 2010-2018 Google, Inc. http://angularjs.org
 License: MIT
*/
'use strict';(function(l,e){'use strict';function B(e){u&&e.get("$route")}function C(x,v,h){return{restrict:"ECA",terminal:!0,priority:400,transclude:"element",link:function(a,b,d,f,q){function m(){k&&(h.cancel(k),k=null);n&&(n.$destroy(),n=null);p&&(k=h.leave(p),k.done(function(a){!1!==a&&(k=null)}),p=null)}function y(){var d=x.current&&x.current.locals;if(e.isDefined(d&&d.$template)){d=a.$new();var f=x.current;p=q(d,function(d){h.enter(d,null,p||b).done(function(b){!1===b||!e.isDefined(z)||z&&!a.$eval(z)||v()});
m()});n=f.scope=d;n.$emit("$viewContentLoaded");n.$eval(l)}else m()}var n,p,k,z=d.autoscroll,l=d.onload||"";a.$on("$routeChangeSuccess",y);y()}}}function A(e,l,h){return{restrict:"ECA",priority:-400,link:function(a,b){var d=h.current,f=d.locals;b.html(f.$template);var q=e(b.contents());if(d.controller){f.$scope=a;var m=l(d.controller,f);d.controllerAs&&(a[d.controllerAs]=m);b.data("$ngControllerController",m);b.children().data("$ngControllerController",m)}a[d.resolveAs||"$resolve"]=f;q(a)}}}"use strict";
var E,F,G,H;l=e.module("ngRoute",[]).info({angularVersion:"1.6.4-local+sha.617b36117"}).provider("$route",function(){function l(a,b){return e.extend(Object.create(a),b)}function v(a,b){b=b.caseInsensitiveMatch;var e={originalPath:a,regexp:a},f=e.keys=[];a=a.replace(/([().])/g,"\\$1").replace(/(\/)?:(\w+)(\*\?|[?*])?/g,function(a,e,b,d){a="?"===d||"*?"===d?"?":null;d="*"===d||"*?"===d?"*":null;f.push({name:b,optional:!!a});e=e||"";return""+(a?"":e)+"(?:"+(a?e:"")+(d&&"(.+?)"||"([^/]+)")+(a||"")+")"+
(a||"")}).replace(/([/$*])/g,"\\$1");e.regexp=new RegExp("^"+a+"$",b?"i":"");return e}E=e.isArray;F=e.isObject;G=e.isDefined;H=e.noop;var h={};this.when=function(a,b){var d=void 0;if(E(b)){d=d||[];for(var f=0,l=b.length;f<l;f++)d[f]=b[f]}else if(F(b))for(f in d=d||{},b)if("$"!==f.charAt(0)||"$"!==f.charAt(1))d[f]=b[f];b=d||b;e.isUndefined(b.reloadOnSearch)&&(b.reloadOnSearch=!0);e.isUndefined(b.caseInsensitiveMatch)&&(b.caseInsensitiveMatch=this.caseInsensitiveMatch);h[a]=e.extend(b,a&&v(a,b));a&&
(d="/"===a[a.length-1]?a.substr(0,a.length-1):a+"/",h[d]=e.extend({redirectTo:a},v(d,b)));return this};this.caseInsensitiveMatch=!1;this.otherwise=function(a){"string"===typeof a&&(a={redirectTo:a});this.when(null,a);return this};u=!0;this.eagerInstantiationEnabled=function(a){return G(a)?(u=a,this):u};this.$get=["$rootScope","$location","$routeParams","$q","$injector","$templateRequest","$sce","$browser",function(a,b,d,f,q,m,u,n){function p(c){var g=r.current;(A=(t=C())&&g&&t.$$route===g.$$route&&
e.equals(t.pathParams,g.pathParams)&&!t.reloadOnSearch&&!D)||!g&&!t||a.$broadcast("$routeChangeStart",t,g).defaultPrevented&&c&&c.preventDefault()}function k(){var c=r.current,g=t;if(A)c.params=g.params,e.copy(c.params,d),a.$broadcast("$routeUpdate",c);else if(g||c){D=!1;r.current=g;var b=f.resolve(g);n.$$incOutstandingRequestCount();b.then(z).then(v).then(function(w){return w&&b.then(x).then(function(b){g===r.current&&(g&&(g.locals=b,e.copy(g.params,d)),a.$broadcast("$routeChangeSuccess",g,c))})}).catch(function(b){g===
r.current&&a.$broadcast("$routeChangeError",g,c,b)}).finally(function(){n.$$completeOutstandingRequest(H)})}}function z(c){var a={route:c,hasRedirection:!1};if(c)if(c.redirectTo)if(e.isString(c.redirectTo))a.path=y(c.redirectTo,c.params),a.search=c.params,a.hasRedirection=!0;else{var d=b.path(),w=b.search();c=c.redirectTo(c.pathParams,d,w);e.isDefined(c)&&(a.url=c,a.hasRedirection=!0)}else if(c.resolveRedirectTo)return f.resolve(q.invoke(c.resolveRedirectTo)).then(function(c){e.isDefined(c)&&(a.url=
c,a.hasRedirection=!0);return a});return a}function v(a){var c=!0;if(a.route!==r.current)c=!1;else if(a.hasRedirection){var e=b.url(),d=a.url;d?b.url(d).replace():d=b.path(a.path).search(a.search).replace().url();d!==e&&(c=!1)}return c}function x(a){if(a){var c=e.extend({},a.resolve);e.forEach(c,function(a,b){c[b]=e.isString(a)?q.get(a):q.invoke(a,null,null,b)});a=B(a);e.isDefined(a)&&(c.$template=a);return f.all(c)}}function B(a){var b,c;e.isDefined(b=a.template)?e.isFunction(b)&&(b=b(a.params)):
e.isDefined(c=a.templateUrl)&&(e.isFunction(c)&&(c=c(a.params)),e.isDefined(c)&&(a.loadedTemplateUrl=u.valueOf(c),b=m(c)));return b}function C(){var a,d;e.forEach(h,function(c,f){if(f=!d){var g=b.path();f=c.keys;var h={};if(c.regexp)if(g=c.regexp.exec(g)){for(var k=1,n=g.length;k<n;++k){var w=f[k-1],m=g[k];w&&m&&(h[w.name]=m)}f=h}else f=null;else f=null;f=a=f}f&&(d=l(c,{params:e.extend({},b.search(),a),pathParams:a}),d.$$route=c)});return d||h[null]&&l(h[null],{params:{},pathParams:{}})}function y(a,
b){var c=[];e.forEach((a||"").split(":"),function(a,d){0===d?c.push(a):(a=a.match(/(\w+)(?:[?*])?(.*)/),d=a[1],c.push(b[d]),c.push(a[2]||""),delete b[d])});return c.join("")}var D=!1,t,A,r={routes:h,reload:function(){D=!0;var c={defaultPrevented:!1,preventDefault:function(){this.defaultPrevented=!0;D=!1}};a.$evalAsync(function(){p(c);c.defaultPrevented||k()})},updateParams:function(a){if(this.current&&this.current.$$route)a=e.extend({},this.current.params,a),b.path(y(this.current.$$route.originalPath,
a)),b.search(a);else throw I("norout");}};a.$on("$locationChangeStart",p);a.$on("$locationChangeSuccess",k);return r}]}).run(B);var I=e.$$minErr("ngRoute"),u;B.$inject=["$injector"];"use strict";l.provider("$routeParams",function(){this.$get=function(){return{}}});"use strict";l.directive("ngView",C);l.directive("ngView",A);C.$inject=["$route","$anchorScroll","$animate"];A.$inject=["$compile","$controller","$route"]})(window,window.angular);

//third_party/javascript/angular/v1_6/angular-sanitize.min.js
/*
 AngularJS v1.6.4-local+sha.617b36117
 (c) 2010-2018 Google, Inc. http://angularjs.org
 License: MIT
*/
'use strict';(function(r,b){'use strict';function Q(b){var g=[];C(g,F).chars(b);return g.join("")}var D=b.$$minErr("$sanitize"),G,h,H,I,J,p,F,K,L,C;b.module("ngSanitize",[]).provider("$sanitize",function(){function g(a,c){return B(a.split(","),c)}function B(a,c){var d={},e;for(e=0;e<a.length;e++)d[c?p(a[e]):a[e]]=!0;return d}function t(a,c){c&&c.length&&h(a,B(c))}function R(a){for(var c={},d=0,e=a.length;d<e;d++){var q=a[d];c[q.name]=q.value}return c}function M(a){return a.replace(/&/g,"&amp;").replace(z,function(a){var c=
a.charCodeAt(0);a=a.charCodeAt(1);return"&#"+(1024*(c-55296)+(a-56320)+65536)+";"}).replace(u,function(a){return"&#"+a.charCodeAt(0)+";"}).replace(/</g,"&lt;").replace(/>/g,"&gt;")}function A(a){for(;a;){if(a.nodeType===r.Node.ELEMENT_NODE)for(var c=a.attributes,d=0,e=c.length;d<e;d++){var q=c[d],f=q.name.toLowerCase();if("xmlns:ns1"===f||0===f.lastIndexOf("ns1:",0))a.removeAttributeNode(q),d--,e--}(c=a.firstChild)&&A(c);a=v("nextSibling",a)}}function v(a,c){if((a=c[a])&&K.call(c,a))throw D("elclob",
c.outerHTML||c.outerText);return a}var y=!1,f=!1;this.$get=["$$sanitizeUri",function(a){y=!0;f&&h(l,k);return function(c){var d=[];L(c,C(d,function(c,d){return!/^unsafe:/.test(a(c,d))}));return d.join("")}}];this.enableSvg=function(a){return J(a)?(f=a,this):f};this.addValidElements=function(a){y||(I(a)&&(a={htmlElements:a}),t(k,a.svgElements),t(E,a.htmlVoidElements),t(l,a.htmlVoidElements),t(l,a.htmlElements));return this};this.addValidAttrs=function(a){y||h(N,B(a,!0));return this};G=b.bind;h=b.extend;
H=b.forEach;I=b.isArray;J=b.isDefined;p=b.lowercase;F=b.noop;L=function(a,c){null===a||void 0===a?a="":"string"!==typeof a&&(a=""+a);var d=O(a);if(!d)return"";var e=5;do{if(0===e)throw D("uinput");e--;a=d.innerHTML;d=O(a)}while(a!==d.innerHTML);for(a=d.firstChild;a;){switch(a.nodeType){case 1:c.start(a.nodeName.toLowerCase(),R(a.attributes));break;case 3:c.chars(a.textContent)}if(!(e=a.firstChild)&&(1===a.nodeType&&c.end(a.nodeName.toLowerCase()),e=v("nextSibling",a),!e))for(;null==e;){a=v("parentNode",
a);if(a===d)break;e=v("nextSibling",a);1===a.nodeType&&c.end(a.nodeName.toLowerCase())}a=e}for(;a=d.firstChild;)d.removeChild(a)};C=function(a,c){var d=!1,e=G(a,a.push);return{start:function(a,f){a=p(a);!d&&w[a]&&(d=a);d||!0!==l[a]||(e("<"),e(a),H(f,function(d,f){var b=p(f),q="img"===a&&"src"===b||"background"===b;!0!==N[b]||!0===P[b]&&!c(d,q)||(e(" "),e(f),e('="'),e(M(d)),e('"'))}),e(">"))},end:function(a){a=p(a);d||!0!==l[a]||!0===E[a]||(e("</"),e(a),e(">"));a==d&&(d=!1)},chars:function(a){d||e(M(a))}}};
K=r.Node.prototype.contains||function(a){return!!(this.compareDocumentPosition(a)&16)};var z=/[\uD800-\uDBFF][\uDC00-\uDFFF]/g,u=/([^#-~ |!])/g,E=g("area,br,col,hr,img,wbr"),x=g("colgroup,dd,dt,li,p,tbody,td,tfoot,th,thead,tr"),n=g("rp,rt"),m=h({},n,x);x=h({},x,g("address,article,aside,blockquote,caption,center,del,dir,div,dl,figure,figcaption,footer,h1,h2,h3,h4,h5,h6,header,hgroup,hr,ins,map,menu,nav,ol,pre,section,table,ul"));n=h({},n,g("a,abbr,acronym,b,bdi,bdo,big,br,cite,code,del,dfn,em,font,i,img,ins,kbd,label,map,mark,q,ruby,rp,rt,s,samp,small,span,strike,strong,sub,sup,time,tt,u,var"));
var k=g("circle,defs,desc,ellipse,font-face,font-face-name,font-face-src,g,glyph,hkern,image,linearGradient,line,marker,metadata,missing-glyph,mpath,path,polygon,polyline,radialGradient,rect,stop,svg,switch,text,title,tspan"),w=g("script,style"),l=h({},E,x,n,m),P=g("background,cite,href,longdesc,src,xlink:href,xml:base");m=g("abbr,align,alt,axis,bgcolor,border,cellpadding,cellspacing,class,clear,color,cols,colspan,compact,coords,dir,face,headers,height,hreflang,hspace,ismap,lang,language,nohref,nowrap,rel,rev,rows,rowspan,rules,scope,scrolling,shape,size,span,start,summary,tabindex,target,title,type,valign,value,vspace,width");
n=g("accent-height,accumulate,additive,alphabetic,arabic-form,ascent,baseProfile,bbox,begin,by,calcMode,cap-height,class,color,color-rendering,content,cx,cy,d,dx,dy,descent,display,dur,end,fill,fill-rule,font-family,font-size,font-stretch,font-style,font-variant,font-weight,from,fx,fy,g1,g2,glyph-name,gradientUnits,hanging,height,horiz-adv-x,horiz-origin-x,ideographic,k,keyPoints,keySplines,keyTimes,lang,marker-end,marker-mid,marker-start,markerHeight,markerUnits,markerWidth,mathematical,max,min,offset,opacity,orient,origin,overline-position,overline-thickness,panose-1,path,pathLength,points,preserveAspectRatio,r,refX,refY,repeatCount,repeatDur,requiredExtensions,requiredFeatures,restart,rotate,rx,ry,slope,stemh,stemv,stop-color,stop-opacity,strikethrough-position,strikethrough-thickness,stroke,stroke-dasharray,stroke-dashoffset,stroke-linecap,stroke-linejoin,stroke-miterlimit,stroke-opacity,stroke-width,systemLanguage,target,text-anchor,to,transform,type,u1,u2,underline-position,underline-thickness,unicode,unicode-range,units-per-em,values,version,viewBox,visibility,width,widths,x,x-height,x1,x2,xlink:actuate,xlink:arcrole,xlink:role,xlink:show,xlink:title,xlink:type,xml:base,xml:lang,xml:space,xmlns,xmlns:xlink,y,y1,y2,zoomAndPan",
!0);var N=h({},P,n,m),O=function(a,c){function d(c){c="<remove></remove>"+c;try{var d=(new a.DOMParser).parseFromString(c,"text/html").body;d.firstChild.remove();return d}catch(S){}}function e(a){b.innerHTML=a;c.documentMode&&A(b);return b}if(c&&c.implementation)var f=c.implementation.createHTMLDocument("inert");else throw D("noinert");var b=(f.documentElement||f.getDocumentElement()).querySelector("body");b.innerHTML='<svg><g onload="this.parentNode.remove()"></g></svg>';return b.querySelector("svg")?
(b.innerHTML='<svg><p><style><img src="</style><img src=x onerror=alert(1)//">',b.querySelector("svg img")?d:e):function(c){c="<remove></remove>"+c;try{c=encodeURI(c)}catch(S){return}var d=new a.XMLHttpRequest;d.responseType="document";d.open("GET","data:text/html;charset=utf-8,"+c,!1);d.send(null);c=d.response.body;c.firstChild.remove();return c}}(r,r.document)}).info({angularVersion:"1.6.4-local+sha.617b36117"});b.module("ngSanitize").filter("linky",["$sanitize",function(g){var h=/((s?ftp|https?):\/\/|(www\.)|(mailto:)?[A-Za-z0-9._%+-]+@)\S*[^\s.;,(){}<>"\u201d\u2019]/i,
t=/^mailto:/i,p=b.$$minErr("linky"),r=b.isDefined,A=b.isFunction,v=b.isObject,y=b.isString;return function(b,z,u){function f(b){b&&k.push(Q(b))}function x(b,g){var h,a=n(b);k.push("<a ");for(h in a)k.push(h+'="'+a[h]+'" ');!r(z)||"target"in a||k.push('target="',z,'" ');k.push('href="',b.replace(/"/g,"&quot;"),'">');f(g);k.push("</a>")}if(null==b||""===b)return b;if(!y(b))throw p("notstring",b);for(var n=A(u)?u:v(u)?function(){return u}:function(){return{}},m=b,k=[],w,l;b=m.match(h);)w=b[0],b[2]||
b[4]||(w=(b[3]?"http://":"mailto:")+w),l=b.index,f(m.substr(0,l)),x(w,b[0].replace(t,"")),m=m.substring(l+b[0].length);f(m);return g(k.join(""))}}])})(window,window.angular);

//third_party/javascript/angular/v1_6/angular-animate.min.js
/*
 AngularJS v1.6.4-local+sha.617b36117
 (c) 2010-2018 Google, Inc. http://angularjs.org
 License: MIT
*/
'use strict';(function(I,y){'use strict';function Ca(a,b,c){if(!a)throw Qa("areq",b||"?",c||"required");return a}function Da(a,b){if(!a&&!b)return"";if(!a)return b;if(!b)return a;S(a)&&(a=a.join(" "));S(b)&&(b=b.join(" "));return a+" "+b}function Ra(a){var b={};a&&(a.to||a.from)&&(b.to=a.to,b.from=a.from);return b}function T(a,b,c){var d="";a=S(a)?a:a&&D(a)&&a.length?a.split(/\s+/):[];u(a,function(a,m){a&&0<a.length&&(d+=0<m?" ":"",d+=c?b+a:a+b)});return d}function Ea(a){if(a instanceof B)switch(a.length){case 0:return a;
case 1:if(1===a[0].nodeType)return a;break;default:return B(ua(a))}if(1===a.nodeType)return B(a)}function ua(a){if(!a[0])return a;for(var b=0;b<a.length;b++){var c=a[b];if(1===c.nodeType)return c}}function Sa(a,b,c){u(b,function(b){a.addClass(b,c)})}function Ta(a,b,c){u(b,function(b){a.removeClass(b,c)})}function U(a){return function(b,c){c.addClass&&(Sa(a,b,c.addClass),c.addClass=null);c.removeClass&&(Ta(a,b,c.removeClass),c.removeClass=null)}}function oa(a){a=a||{};if(!a.$$prepared){var b=a.domOperation||
M;a.domOperation=function(){a.$$domOperationFired=!0;b();b=M};a.$$prepared=!0}return a}function ha(a,b){Fa(a,b);Ga(a,b)}function Fa(a,b){b.from&&(a.css(b.from),b.from=null)}function Ga(a,b){b.to&&(a.css(b.to),b.to=null)}function V(a,b,c){var d=b.options||{};c=c.options||{};var g=(d.addClass||"")+" "+(c.addClass||""),m=(d.removeClass||"")+" "+(c.removeClass||"");a=Ua(a.attr("class"),g,m);c.preparationClasses&&(d.preparationClasses=W(c.preparationClasses,d.preparationClasses),delete c.preparationClasses);
g=d.domOperation!==M?d.domOperation:null;va(d,c);g&&(d.domOperation=g);d.addClass=a.addClass?a.addClass:null;d.removeClass=a.removeClass?a.removeClass:null;b.addClass=d.addClass;b.removeClass=d.removeClass;return d}function Ua(a,b,c){function d(a){D(a)&&(a=a.split(" "));var b={};u(a,function(a){a.length&&(b[a]=!0)});return b}var g={};a=d(a);b=d(b);u(b,function(a,b){g[b]=1});c=d(c);u(c,function(a,b){g[b]=1===g[b]?null:-1});var m={addClass:"",removeClass:""};u(g,function(b,c){if(1===b){var k="addClass";
var d=!a[c]||a[c+"-remove"]}else-1===b&&(k="removeClass",d=a[c]||a[c+"-add"]);d&&(m[k].length&&(m[k]+=" "),m[k]+=c)});return m}function N(a){return a instanceof B?a[0]:a}function Va(a,b,c){var d="";b&&(d=T(b,"ng-",!0));c.addClass&&(d=W(d,T(c.addClass,"-add")));c.removeClass&&(d=W(d,T(c.removeClass,"-remove")));d.length&&(c.preparationClasses=d,a.addClass(d))}function pa(a,b){b=b?"-"+b+"s":"";ia(a,[ja,b]);return[ja,b]}function wa(a,b){b=b?"paused":"";var c=Z+"PlayState";ia(a,[c,b]);return[c,b]}function ia(a,
b){a.style[b[0]]=b[1]}function W(a,b){return a?b?a+" "+b:a:b}function Ha(a,b,c){var d=Object.create(null),g=a.getComputedStyle(b)||{};u(c,function(a,b){if(a=g[a]){var c=a.charAt(0);if("-"===c||"+"===c||0<=c)a=Wa(a);0===a&&(a=null);d[b]=a}});return d}function Wa(a){var b=0;a=a.split(/\s*,\s*/);u(a,function(a){"s"===a.charAt(a.length-1)&&(a=a.substring(0,a.length-1));a=parseFloat(a)||0;b=b?Math.max(a,b):a});return b}function xa(a){return 0===a||null!=a}function Ia(a,b){var c=R;a+="s";b?c+="Duration":
a+=" linear all";return[c,a]}function Ja(){var a=Object.create(null);return{flush:function(){a=Object.create(null)},count:function(b){return(b=a[b])?b.total:0},get:function(b){return(b=a[b])&&b.value},put:function(b,c){a[b]?a[b].total++:a[b]={total:1,value:c}}}}function Ka(a,b,c){u(c,function(c){a[c]=ya(a[c])?a[c]:b.style.getPropertyValue(c)})}if(void 0===I.ontransitionend&&void 0!==I.onwebkittransitionend){var R="WebkitTransition";var La="webkitTransitionEnd transitionend"}else R="transition",La=
"transitionend";if(void 0===I.onanimationend&&void 0!==I.onwebkitanimationend){var Z="WebkitAnimation";var Ma="webkitAnimationEnd animationend"}else Z="animation",Ma="animationend";var qa=Z+"Delay",za=Z+"Duration",ja=R+"Delay",Na=R+"Duration",Qa=y.$$minErr("ng");"use strict";"use strict";"use strict";var Xa={transitionDuration:Na,transitionDelay:ja,transitionProperty:R+"Property",animationDuration:za,animationDelay:qa,animationIterationCount:Z+"IterationCount"},Ya={transitionDuration:Na,transitionDelay:ja,
animationDuration:za,animationDelay:qa};"use strict";"use strict";"use strict";"use strict";"use strict";"use strict";"use strict";var Aa,va,u,S,ya,aa,Ba,ra,D,P,B,M;y.module("ngAnimate",[],function(){M=y.noop;Aa=y.copy;va=y.extend;B=y.element;u=y.forEach;S=y.isArray;D=y.isString;ra=y.isObject;P=y.isUndefined;ya=y.isDefined;Ba=y.isFunction;aa=y.isElement}).info({angularVersion:"1.6.4-local+sha.617b36117"}).directive("ngAnimateSwap",["$animate","$rootScope",function(a,b){return{restrict:"A",transclude:"element",
terminal:!0,priority:600,link:function(b,d,g,m,k){var c,G;b.$watchCollection(g.ngAnimateSwap||g["for"],function(z){c&&a.leave(c);G&&(G.$destroy(),G=null);if(z||0===z)G=b.$new(),k(G,function(b){c=b;a.enter(b,null,d)})})}}}]).directive("ngAnimateChildren",["$interpolate",function(a){return{link:function(b,c,d){function g(a){c.data("$$ngAnimateChildren","on"===a||"true"===a)}var m=d.ngAnimateChildren;D(m)&&0===m.length?c.data("$$ngAnimateChildren",!0):(g(a(m)(b)),d.$observe("ngAnimateChildren",g))}}}]).factory("$$rAFScheduler",
["$$rAF",function(a){function b(a){g=g.concat(a);c()}function c(){if(g.length){for(var b=g.shift(),k=0;k<b.length;k++)b[k]();d||a(function(){d||c()})}}var d;var g=b.queue=[];b.waitUntilQuiet=function(b){d&&d();d=a(function(){d=null;b();c()})};return b}]).provider("$$animateQueue",["$animateProvider",function(a){function b(a){if(!a)return null;a=a.split(" ");var b=Object.create(null);u(a,function(a){b[a]=!0});return b}function c(a,c){if(a&&c){var k=b(c);return a.split(" ").some(function(a){return k[a]})}}
function d(a,b,c){return m[a].some(function(a){return a(b,c)})}function g(a,b){var c=0<(a.addClass||"").length;a=0<(a.removeClass||"").length;return b?c&&a:c||a}var m=this.rules={skip:[],cancel:[],join:[]};m.join.push(function(a,b){return!a.structural&&g(a)});m.skip.push(function(a,b){return!a.structural&&!g(a)});m.skip.push(function(a,b){return"leave"===b.event&&a.structural});m.skip.push(function(a,b){return b.structural&&2===b.state&&!a.structural});m.cancel.push(function(a,b){return b.structural&&
a.structural});m.cancel.push(function(a,b){return 2===b.state&&a.structural});m.cancel.push(function(a,b){if(b.structural)return!1;var d=a.addClass;a=a.removeClass;var g=b.addClass;b=b.removeClass;return P(d)&&P(a)||P(g)&&P(b)?!1:c(d,b)||c(a,g)});this.$get=["$$rAF","$rootScope","$rootElement","$document","$$Map","$$animation","$$AnimateRunner","$templateRequest","$$jqLite","$$forceReflow","$$isDocumentHidden",function(b,c,m,v,Q,K,ka,E,l,J,O){function k(){var a=!1;return function(b){a?b():c.$$postDigest(function(){a=
!0;b()})}}function ba(a,b,c){var e=[],d=h[c];d&&u(d,function(h){z.call(h.node,b)?e.push(h.callback):"leave"===c&&z.call(h.node,a)&&e.push(h.callback)});return e}function f(a,b,c){var e=ua(b);return a.filter(function(a){return!(a.node===e&&(!c||a.callback===c))})}function n(a,h,f){function sa(a,c,e,h){v(function(){var a=ba(m,x,c);a.length?b(function(){u(a,function(a){a(l,e,h)});"close"!==e||x.parentNode||ta.off(x)}):"close"!==e||x.parentNode||ta.off(x)});a.progress(c,e,h)}function n(a){var b=l,c=p;
c.preparationClasses&&(b.removeClass(c.preparationClasses),c.preparationClasses=null);c.activeClasses&&(b.removeClass(c.activeClasses),c.activeClasses=null);Oa(l,p);ha(l,p);p.domOperation();w.complete(!a)}var p=Aa(f),l=Ea(a),x=N(l),m=x&&x.parentNode;p=oa(p);var w=new ka,v=k();S(p.addClass)&&(p.addClass=p.addClass.join(" "));p.addClass&&!D(p.addClass)&&(p.addClass=null);S(p.removeClass)&&(p.removeClass=p.removeClass.join(" "));p.removeClass&&!D(p.removeClass)&&(p.removeClass=null);p.from&&!ra(p.from)&&
(p.from=null);p.to&&!ra(p.to)&&(p.to=null);if(!(e&&x&&Za(x,h,f)&&F(x,p)))return n(),w;var J=0<=["enter","move","leave"].indexOf(h),q=O(),A=q||ea.get(x);f=!A&&H.get(x)||{};var la=!!f.state;A||la&&1===f.state||(A=!fa(x,m,h));if(A)return q&&sa(w,h,"start"),n(),q&&sa(w,h,"close"),w;J&&G(x);q={structural:J,element:l,event:h,addClass:p.addClass,removeClass:p.removeClass,close:n,options:p,runner:w};if(la){if(d("skip",q,f)){if(2===f.state)return n(),w;V(l,f,q);return f.runner}if(d("cancel",q,f))if(2===f.state)f.runner.end();
else if(f.structural)f.close();else return V(l,f,q),f.runner;else if(d("join",q,f))if(2===f.state)V(l,q,{});else return Va(l,J?h:null,p),h=q.event=f.event,p=V(l,f,q),f.runner}else V(l,q,{});(la=q.structural)||(la="animate"===q.event&&0<Object.keys(q.options.to||{}).length||g(q));if(!la)return n(),t(x),w;var z=(f.counter||0)+1;q.counter=z;C(x,1,q);c.$$postDigest(function(){l=Ea(a);var b=H.get(x),c=!b;b=b||{};var e=0<(l.parent()||[]).length&&("animate"===b.event||b.structural||g(b));if(c||b.counter!==
z||!e){c&&(Oa(l,p),ha(l,p));if(c||J&&b.event!==h)p.domOperation(),w.end();e||t(x)}else h=!b.structural&&g(b,!0)?"setClass":b.event,C(x,2),b=K(l,h,b.options),w.setHost(b),sa(w,h,"start",{}),b.done(function(a){n(!a);(a=H.get(x))&&a.counter===z&&t(x);sa(w,h,"close",{})})});return w}function G(a){a=a.querySelectorAll("[data-ng-animate]");u(a,function(a){var b=parseInt(a.getAttribute("data-ng-animate"),10),c=H.get(a);if(c)switch(b){case 2:c.runner.end();case 1:H.delete(a)}})}function t(a){a.removeAttribute("data-ng-animate");
H.delete(a)}function fa(a,b,c){c=v[0].body;var e=N(m),h=a===c||"HTML"===a.nodeName,f=a===e,d=!1,p=ea.get(a),l;for((a=B.data(a,"$ngAnimatePin"))&&(b=N(a));b;){f||(f=b===e);if(1!==b.nodeType)break;a=H.get(b)||{};if(!d){var n=ea.get(b);if(!0===n&&!1!==p){p=!0;break}else!1===n&&(p=!1);d=a.structural}if(P(l)||!0===l)a=B.data(b,"$$ngAnimateChildren"),ya(a)&&(l=a);if(d&&!1===l)break;h||(h=b===c);if(h&&f)break;if(!f&&(a=B.data(b,"$ngAnimatePin"))){b=N(a);continue}b=b.parentNode}return(!d||l)&&!0!==p&&f&&
h}function C(a,b,c){c=c||{};c.state=b;a.setAttribute("data-ng-animate",b);c=(b=H.get(a))?va(b,c):c;H.set(a,c)}var H=new Q,ea=new Q,e=null,x=c.$watch(function(){return 0===E.totalPendingRequests},function(a){a&&(x(),c.$$postDigest(function(){c.$$postDigest(function(){null===e&&(e=!0)})}))}),h=Object.create(null);Q=a.customFilter();var w=a.classNameFilter();J=function(){return!0};var Za=Q||J,F=w?function(a,b){a=[a.getAttribute("class"),b.addClass,b.removeClass].join(" ");return w.test(a)}:J,Oa=U(l),
z=I.Node.prototype.contains||function(a){return this===a||!!(this.compareDocumentPosition(a)&16)},ta={on:function(a,b,c){var e=ua(b);h[a]=h[a]||[];h[a].push({node:e,callback:c});B(b).on("$destroy",function(){H.get(e)||ta.off(a,b,c)})},off:function(a,b,c){if(1!==arguments.length||D(arguments[0])){var e=h[a];e&&(h[a]=1===arguments.length?null:f(e,b,c))}else for(e in b=arguments[0],h)h[e]=f(h[e],b)},pin:function(a,b){Ca(aa(a),"element","not an element");Ca(aa(b),"parentElement","not an element");a.data("$ngAnimatePin",
b)},push:function(a,b,c,e){c=c||{};c.domOperation=e;return n(a,b,c)},enabled:function(a,b){var c=arguments.length;if(0===c)b=!!e;else if(aa(a)){var h=N(a);1===c?b=!ea.get(h):ea.set(h,!b)}else b=e=!!a;return b}};return ta}]}]).provider("$$animation",["$animateProvider",function(a){var b=this.drivers=[];this.$get=["$$jqLite","$rootScope","$injector","$$AnimateRunner","$$Map","$$rAFScheduler",function(a,d,g,m,k,z){function c(a){function b(a){if(a.processed)return a;a.processed=!0;var d=a.domNode,f=d.parentNode;
g.set(d,a);for(var n;f;){if(n=g.get(f)){n.processed||(n=b(n));break}f=f.parentNode}(n||c).children.push(a);return a}var c={children:[]},d,g=new k;for(d=0;d<a.length;d++){var m=a[d];g.set(m.domNode,a[d]={domNode:m.domNode,fn:m.fn,children:[]})}for(d=0;d<a.length;d++)b(a[d]);return function(a){var b=[],c=[],d;for(d=0;d<a.children.length;d++)c.push(a.children[d]);a=c.length;var l=0,g=[];for(d=0;d<c.length;d++){var m=c[d];0>=a&&(a=l,l=0,b.push(g),g=[]);g.push(m.fn);m.children.forEach(function(a){l++;
c.push(a)});a--}g.length&&b.push(g);return b}(c)}var v=[],Q=U(a);return function(k,G,E){function l(a){a=a.hasAttribute("ng-animate-ref")?[a]:a.querySelectorAll("[ng-animate-ref]");var b=[];u(a,function(a){var c=a.getAttribute("ng-animate-ref");c&&c.length&&b.push(a)});return b}function J(a){var b=[],c={};u(a,function(a,e){var d=N(a.element),h=0<=["enter","move"].indexOf(a.event);d=a.structural?l(d):[];if(d.length){var f=h?"to":"from";u(d,function(a){var b=a.getAttribute("ng-animate-ref");c[b]=c[b]||
{};c[b][f]={animationID:e,element:B(a)}})}else b.push(a)});var h={},d={};u(c,function(c,e){e=c.from;c=c.to;if(e&&c){var f=a[e.animationID],l=a[c.animationID],g=e.animationID.toString();if(!d[g]){var n=d[g]={structural:!0,beforeStart:function(){f.beforeStart();l.beforeStart()},close:function(){f.close();l.close()},classes:O(f.classes,l.classes),from:f,to:l,anchors:[]};n.classes.length?b.push(n):(b.push(f),b.push(l))}d[g].anchors.push({out:e.element,"in":c.element})}else e=e?e.animationID:c.animationID,
c=e.toString(),h[c]||(h[c]=!0,b.push(a[e]))});return b}function O(a,b){a=a.split(" ");b=b.split(" ");for(var c=[],e=0;e<a.length;e++){var d=a[e];if("ng-"!==d.substring(0,3))for(var f=0;f<b.length;f++)if(d===b[f]){c.push(d);break}}return c.join(" ")}function A(a){for(var c=b.length-1;0<=c;c--){var d=g.get(b[c])(a);if(d)return d}}function ba(a,b){function c(a){(a=a.data("$$animationRunner"))&&a.setHost(b)}a.from&&a.to?(c(a.from.element),c(a.to.element)):c(a.element)}function f(){var a=k.data("$$animationRunner");
!a||"leave"===G&&E.$$domOperationFired||a.end()}function n(b){k.off("$destroy",f);k.removeData("$$animationRunner");Q(k,E);ha(k,E);E.domOperation();C&&a.removeClass(k,C);k.removeClass("ng-animate");t.complete(!b)}E=oa(E);var Pa=0<=["enter","move","leave"].indexOf(G),t=new m({end:function(){n()},cancel:function(){n(!0)}});if(!b.length)return n(),t;k.data("$$animationRunner",t);var fa=Da(k.attr("class"),Da(E.addClass,E.removeClass)),C=E.tempClasses;C&&(fa+=" "+C,E.tempClasses=null);if(Pa){var H="ng-"+
G+"-prepare";a.addClass(k,H)}v.push({element:k,classes:fa,event:G,structural:Pa,options:E,beforeStart:function(){k.addClass("ng-animate");C&&a.addClass(k,C);H&&(a.removeClass(k,H),H=null)},close:n});k.on("$destroy",f);if(1<v.length)return t;d.$$postDigest(function(){var a=[];u(v,function(b){b.element.data("$$animationRunner")?a.push(b):b.close()});v.length=0;var b=J(a),d=[];u(b,function(a){d.push({domNode:N(a.from?a.from.element:a.element),fn:function(){a.beforeStart();var b=a.close;if((a.anchors?
a.from.element||a.to.element:a.element).data("$$animationRunner")){var c=A(a);if(c)var e=c.start}e?(e=e(),e.done(function(a){b(!a)}),ba(a,e)):b()}})});z(c(d))});return t}}]}]).provider("$animateCss",["$animateProvider",function(a){var b=Ja(),c=Ja();this.$get=["$window","$$jqLite","$$AnimateRunner","$timeout","$$forceReflow","$sniffer","$$rAFScheduler","$$animateQueue",function(a,g,m,k,z,G,v,Q){function d(a,b){var c=a.parentNode;return(c.$$ngAnimateParentKey||(c.$$ngAnimateParentKey=++O))+"-"+a.getAttribute("class")+
"-"+b}function y(d,f,l,k){if(0<b.count(l)){var n=c.get(l);n||(f=T(f,"-stagger"),g.addClass(d,f),n=Ha(a,d,k),n.animationDuration=Math.max(n.animationDuration,0),n.transitionDuration=Math.max(n.transitionDuration,0),g.removeClass(d,f),c.put(l,n))}return n||{}}function E(a){A.push(a);v.waitUntilQuiet(function(){b.flush();c.flush();for(var a=z(),d=0;d<A.length;d++)A[d](a);A.length=0})}function l(c,d,l){d=b.get(l);d||(d=Ha(a,c,Xa),"infinite"===d.animationIterationCount&&(d.animationIterationCount=1));
b.put(l,d);c=d;l=c.animationDelay;d=c.transitionDelay;c.maxDelay=l&&d?Math.max(l,d):l||d;c.maxDuration=Math.max(c.animationDuration*c.animationIterationCount,c.transitionDuration);return c}var J=U(g),O=0,A=[];return function(a,c){function f(){t()}function v(){t(!0)}function t(b){if(!(O||ba&&A)){O=!0;A=!1;e.$$skipPreparationClasses||g.removeClass(a,da);g.removeClass(a,V);wa(h,!1);pa(h,!1);u(w,function(a){h.style[a[0]]=""});J(a,e);ha(a,e);Object.keys(x).length&&u(x,function(a,b){a?h.style.setProperty(b,
a):h.style.removeProperty(b)});if(e.onDone)e.onDone();K&&K.length&&a.off(K.join(" "),H);var c=a.data("$$animateCss");c&&(k.cancel(c[0].timer),a.removeData("$$animateCss"));B&&B.complete(!b)}}function fa(a){r.blockTransition&&pa(h,a);r.blockKeyframeAnimation&&wa(h,!!a)}function C(){B=new m({end:f,cancel:v});E(M);t();return{$$willAnimate:!1,start:function(){return B},end:f}}function H(a){a.stopPropagation();var b=a.originalEvent||a;b.target===h&&(a=b.$manualTimeStamp||Date.now(),b=parseFloat(b.elapsedTime.toFixed(3)),
Math.max(a-ka,0)>=W&&b>=L&&(ba=!0,t()))}function ea(){function b(){if(!O){fa(!1);u(w,function(a){h.style[a[0]]=a[1]});J(a,e);g.addClass(a,V);if(r.recalculateTimingStyles){I=h.getAttribute("class")+" "+da;ma=d(h,I);q=l(h,I,ma);Y=q.maxDelay;na=Math.max(Y,0);L=q.maxDuration;if(0===L){t();return}r.hasTransitions=0<q.transitionDuration;r.hasAnimations=0<q.animationDuration}r.applyAnimationDelay&&(Y="boolean"!==typeof e.delay&&xa(e.delay)?parseFloat(e.delay):Y,na=Math.max(Y,0),q.animationDelay=Y,ca=[qa,
Y+"s"],w.push(ca),h.style[ca[0]]=ca[1]);W=1E3*na;aa=1E3*L;if(e.easing){var b=e.easing;if(r.hasTransitions){var f=R+"TimingFunction";w.push([f,b]);h.style[f]=b}r.hasAnimations&&(f=Z+"TimingFunction",w.push([f,b]),h.style[f]=b)}q.transitionDuration&&K.push(La);q.animationDuration&&K.push(Ma);ka=Date.now();var C=W+1.5*aa;f=ka+C;b=a.data("$$animateCss")||[];var m=!0;if(b.length){var n=b[0];(m=f>n.expectedEndTime)?k.cancel(n.timer):b.push(t)}m&&(C=k(c,C,!1),b[0]={timer:C,expectedEndTime:f},b.push(t),a.data("$$animateCss",
b));if(K.length)a.on(K.join(" "),H);e.to&&(e.cleanupStyles&&Ka(x,h,Object.keys(e.to)),Ga(a,e))}}function c(){var b=a.data("$$animateCss");if(b){for(var c=1;c<b.length;c++)b[c]();a.removeData("$$animateCss")}}if(!O)if(h.parentNode){var f=function(a){if(ba)A&&a&&(A=!1,t());else if(A=!a,q.animationDuration)if(a=wa(h,A),A)w.push(a);else{var b=w,c=b.indexOf(a);0<=a&&b.splice(c,1)}},C=0<U&&(q.transitionDuration&&0===X.transitionDuration||q.animationDuration&&0===X.animationDuration)&&Math.max(X.animationDelay,
X.transitionDelay);C?k(b,Math.floor(C*U*1E3),!1):b();p.resume=function(){f(!0)};p.pause=function(){f(!1)}}else t()}var e=c||{};e.$$prepared||(e=oa(Aa(e)));var x={},h=N(a);if(!h||!h.parentNode||!Q.enabled())return C();var w=[],z=a.attr("class"),F=Ra(e),O,A,ba,B,p,ka,K=[];if(0===e.duration||!G.animations&&!G.transitions)return C();var D=e.event&&S(e.event)?e.event.join(" "):e.event,P="";c="";D&&e.structural?P=T(D,"ng-",!0):D&&(P=D);e.addClass&&(c+=T(e.addClass,"-add"));e.removeClass&&(c.length&&(c+=
" "),c+=T(e.removeClass,"-remove"));e.applyClassesEarly&&c.length&&J(a,e);var da=[P,c].join(" ").trim(),I=z+" "+da,V=T(da,"-active");z=F.to&&0<Object.keys(F.to).length;if(!(0<(e.keyframeStyle||"").length||z||da))return C();if(0<e.stagger){F=parseFloat(e.stagger);var X={transitionDelay:F,animationDelay:F,transitionDuration:0,animationDuration:0}}else{var ma=d(h,I);X=y(h,da,ma,Ya)}e.$$skipPreparationClasses||g.addClass(a,da);e.transitionStyle&&(F=[R,e.transitionStyle],ia(h,F),w.push(F));0<=e.duration&&
(F=0<h.style[R].length,F=Ia(e.duration,F),ia(h,F),w.push(F));e.keyframeStyle&&(F=[Z,e.keyframeStyle],ia(h,F),w.push(F));var U=X?0<=e.staggerIndex?e.staggerIndex:b.count(ma):0;(D=0===U)&&!e.skipBlocking&&pa(h,9999);var q=l(h,I,ma),Y=q.maxDelay;var na=Math.max(Y,0);var L=q.maxDuration;var r={};r.hasTransitions=0<q.transitionDuration;r.hasAnimations=0<q.animationDuration;r.hasTransitionAll=r.hasTransitions&&"all"===q.transitionProperty;r.applyTransitionDuration=z&&(r.hasTransitions&&!r.hasTransitionAll||
r.hasAnimations&&!r.hasTransitions);r.applyAnimationDuration=e.duration&&r.hasAnimations;r.applyTransitionDelay=xa(e.delay)&&(r.applyTransitionDuration||r.hasTransitions);r.applyAnimationDelay=xa(e.delay)&&r.hasAnimations;r.recalculateTimingStyles=0<c.length;if(r.applyTransitionDuration||r.applyAnimationDuration)L=e.duration?parseFloat(e.duration):L,r.applyTransitionDuration&&(r.hasTransitions=!0,q.transitionDuration=L,F=0<h.style[R+"Property"].length,w.push(Ia(L,F))),r.applyAnimationDuration&&(r.hasAnimations=
!0,q.animationDuration=L,w.push([za,L+"s"]));if(0===L&&!r.recalculateTimingStyles)return C();if(null!=e.delay){if("boolean"!==typeof e.delay){var ca=parseFloat(e.delay);na=Math.max(ca,0)}r.applyTransitionDelay&&w.push([ja,ca+"s"]);r.applyAnimationDelay&&w.push([qa,ca+"s"])}null==e.duration&&0<q.transitionDuration&&(r.recalculateTimingStyles=r.recalculateTimingStyles||D);var W=1E3*na;var aa=1E3*L;e.skipBlocking||(r.blockTransition=0<q.transitionDuration,r.blockKeyframeAnimation=0<q.animationDuration&&
0<X.animationDelay&&0===X.animationDuration);e.from&&(e.cleanupStyles&&Ka(x,h,Object.keys(e.from)),Fa(a,e));r.blockTransition||r.blockKeyframeAnimation?fa(L):e.skipBlocking||pa(h,!1);return{$$willAnimate:!0,end:f,start:function(){if(!O)return p={end:f,cancel:v,resume:null,pause:null},B=new m(p),E(ea),B}}}}]}]).provider("$$animateCssDriver",["$$animationProvider",function(a){a.drivers.push("$$animateCssDriver");this.$get=["$animateCss","$rootScope","$$AnimateRunner","$rootElement","$sniffer","$$jqLite",
"$document",function(a,c,d,g,m,k,z){function b(a,b){D(a)&&(a=a.split(" "));D(b)&&(b=b.split(" "));return a.filter(function(a){return-1===b.indexOf(a)}).join(" ")}function v(c,g,k){function l(a){var b={},c=N(a).getBoundingClientRect();u(["width","height","top","left"],function(a){var d=c[a];switch(a){case "top":d+=y.scrollTop;break;case "left":d+=y.scrollLeft}b[a]=Math.floor(d)+"px"});return b}function m(){var c=(k.attr("class")||"").replace(/\bng-\S+\b/g,""),d=b(c,v);c=b(v,c);d=a(n,{to:l(k),addClass:"ng-anchor-in "+
d,removeClass:"ng-anchor-out "+c,delay:!0});return d.$$willAnimate?d:null}function f(){n.remove();g.removeClass("ng-animate-shim");k.removeClass("ng-animate-shim")}var n=B(N(g).cloneNode(!0)),v=(n.attr("class")||"").replace(/\bng-\S+\b/g,"");g.addClass("ng-animate-shim");k.addClass("ng-animate-shim");n.addClass("ng-anchor");E.append(n);c=function(){var b=a(n,{addClass:"ng-anchor-out",delay:!0,from:l(g)});return b.$$willAnimate?b:null}();if(!c){var t=m();if(!t)return f()}var z=c||t;return{start:function(){function a(){c&&
c.end()}var b,c=z.start();c.done(function(){c=null;if(!t&&(t=m()))return c=t.start(),c.done(function(){c=null;f();b.complete()}),c;f();b.complete()});return b=new d({end:a,cancel:a})}}}function Q(a,b,c,g){var l=K(a,M),f=K(b,M),k=[];u(g,function(a){(a=v(c,a.out,a["in"]))&&k.push(a)});if(l||f||0!==k.length)return{start:function(){function a(){u(b,function(a){a.end()})}var b=[];l&&b.push(l.start());f&&b.push(f.start());u(k,function(a){b.push(a.start())});var c=new d({end:a,cancel:a});d.all(b,function(a){c.complete(a)});
return c}}}function K(b){var c=b.element,d=b.options||{};b.structural&&(d.event=b.event,d.structural=!0,d.applyClassesEarly=!0,"leave"===b.event&&(d.onDone=d.domOperation));d.preparationClasses&&(d.event=W(d.event,d.preparationClasses));b=a(c,d);return b.$$willAnimate?b:null}if(!m.animations&&!m.transitions)return M;var y=z[0].body;c=N(g);var E=B(c.parentNode&&11===c.parentNode.nodeType||y.contains(c)?c:y);return function(a){return a.from&&a.to?Q(a.from,a.to,a.classes,a.anchors):K(a)}}]}]).provider("$$animateJs",
["$animateProvider",function(a){this.$get=["$injector","$$AnimateRunner","$$jqLite",function(b,c,d){function g(c){c=S(c)?c:c.split(" ");for(var d=[],g={},k=0;k<c.length;k++){var m=c[k],u=a.$$registeredAnimations[m];u&&!g[m]&&(d.push(b.get(u)),g[m]=!0)}return d}var m=U(d);return function(a,b,d,v){function k(){v.domOperation();m(a,v)}function z(a,b,d,f,e){switch(d){case "animate":b=[b,f.from,f.to,e];break;case "setClass":b=[b,G,B,e];break;case "addClass":b=[b,G,e];break;case "removeClass":b=[b,B,e];
break;default:b=[b,e]}b.push(f);if(a=a.apply(a,b))if(Ba(a.start)&&(a=a.start()),a instanceof c)a.done(e);else if(Ba(a))return a;return M}function y(a,b,d,f,e){var g=[];u(f,function(f){var h=f[e];h&&g.push(function(){var e=!1,f=function(a){e||(e=!0,(k||M)(a),g.complete(!a))};var g=new c({end:function(){f()},cancel:function(){f(!0)}});var k=z(h,a,b,d,function(a){f(!1===a)});return g})});return g}function E(a,b,d,f,e){var g=y(a,b,d,f,e);if(0===g.length){if("beforeSetClass"===e){var h=y(a,"removeClass",
d,f,"beforeRemoveClass");var k=y(a,"addClass",d,f,"beforeAddClass")}else"setClass"===e&&(h=y(a,"removeClass",d,f,"removeClass"),k=y(a,"addClass",d,f,"addClass"));h&&(g=g.concat(h));k&&(g=g.concat(k))}if(0!==g.length)return function(a){var b=[];g.length&&u(g,function(a){b.push(a())});b.length?c.all(b,a):a();return function(a){u(b,function(b){a?b.cancel():b.end()})}}}var l=!1;3===arguments.length&&ra(d)&&(v=d,d=null);v=oa(v);d||(d=a.attr("class")||"",v.addClass&&(d+=" "+v.addClass),v.removeClass&&(d+=
" "+v.removeClass));var G=v.addClass,B=v.removeClass,A=g(d),D;if(A.length){if("leave"===b){var f="leave";var n="afterLeave"}else f="before"+b.charAt(0).toUpperCase()+b.substr(1),n=b;"enter"!==b&&"move"!==b&&(D=E(a,b,v,A,f));var I=E(a,b,v,A,n)}if(D||I){var t;return{$$willAnimate:!0,end:function(){t?t.end():(l=!0,k(),ha(a,v),t=new c,t.complete(!0));return t},start:function(){function b(b){l=!0;k();ha(a,v);t.complete(b)}if(t)return t;t=new c;var d,f=[];D&&f.push(function(a){d=D(a)});f.length?f.push(function(a){k();
a(!0)}):k();I&&f.push(function(a){d=I(a)});t.setHost({end:function(){l||((d||M)(void 0),b(void 0))},cancel:function(){l||((d||M)(!0),b(!0))}});c.chain(f,b);return t}}}}}]}]).provider("$$animateJsDriver",["$$animationProvider",function(a){a.drivers.push("$$animateJsDriver");this.$get=["$$animateJs","$$AnimateRunner",function(a,c){function b(b){return a(b.element,b.event,b.classes,b.options)}return function(a){if(a.from&&a.to){var d=b(a.from),g=b(a.to);return d||g?{start:function(){function a(){return function(){u(b,
function(a){a.end()})}}var b=[];d&&b.push(d.start());g&&b.push(g.start());c.all(b,function(a){k.complete(a)});var k=new c({end:a(),cancel:a()});return k}}:void 0}return b(a)}}]}])})(window,window.angular);

//third_party/javascript/angular/v1_6/angular-aria.min.js
/*
 AngularJS v1.6.4-local+sha.617b36117
 (c) 2010-2018 Google, Inc. http://angularjs.org
 License: MIT
*/
'use strict';(function(t,p){'use strict';var d="BUTTON A INPUT TEXTAREA SELECT DETAILS SUMMARY".split(" "),h=function(a,b){if(-1!==b.indexOf(a[0].nodeName))return!0};p.module("ngAria",["ng"]).info({angularVersion:"1.6.4-local+sha.617b36117"}).provider("$aria",function(){function a(a,d,k,m){return function(e,f,c){var g=c.$normalize(d);!b[g]||h(f,k)||c[g]||e.$watch(c[a],function(a){a=m?!a:!!a;f.attr(d,a)})}}var b={ariaHidden:!0,ariaChecked:!0,ariaReadonly:!0,ariaDisabled:!0,ariaRequired:!0,ariaInvalid:!0,ariaValue:!0,
tabindex:!0,bindKeydown:!0,bindRoleForClick:!0};this.config=function(a){b=p.extend(b,a)};this.$get=function(){return{config:function(a){return b[a]},$$watchExpr:a}}}).directive("ngShow",["$aria",function(a){return a.$$watchExpr("ngShow","aria-hidden",[],!0)}]).directive("ngHide",["$aria",function(a){return a.$$watchExpr("ngHide","aria-hidden",[],!1)}]).directive("ngValue",["$aria",function(a){return a.$$watchExpr("ngValue","aria-checked",d,!1)}]).directive("ngChecked",["$aria",function(a){return a.$$watchExpr("ngChecked",
"aria-checked",d,!1)}]).directive("ngReadonly",["$aria",function(a){return a.$$watchExpr("ngReadonly","aria-readonly",d,!1)}]).directive("ngRequired",["$aria",function(a){return a.$$watchExpr("ngRequired","aria-required",d,!1)}]).directive("ngModel",["$aria",function(a){function b(k,b,e,f){return a.config(b)&&!e.attr(k)&&(f||!h(e,d))}function f(a,b){return!b.attr("role")&&b.attr("type")===a&&!h(b,d)}function l(a,b){b=a.type;a=a.role;return"checkbox"===(b||a)||"menuitemcheckbox"===a?"checkbox":"radio"===
(b||a)||"menuitemradio"===a?"radio":"range"===b||"progressbar"===a||"slider"===a?"range":""}return{restrict:"A",require:"ngModel",priority:200,compile:function(d,m){var e=l(m,d);return{post:function(d,c,g,n){function m(){return n.$modelValue}function h(a){c.attr("aria-checked",g.value==n.$viewValue)}function l(){c.attr("aria-checked",!n.$isEmpty(n.$viewValue))}var k=b("tabindex","tabindex",c,!1);switch(e){case "radio":case "checkbox":f(e,c)&&c.attr("role",e);b("aria-checked","ariaChecked",c,!1)&&
d.$watch(m,"radio"===e?h:l);k&&c.attr("tabindex",0);break;case "range":f(e,c)&&c.attr("role","slider");if(a.config("ariaValue")){var q=!c.attr("aria-valuemin")&&(g.hasOwnProperty("min")||g.hasOwnProperty("ngMin")),p=!c.attr("aria-valuemax")&&(g.hasOwnProperty("max")||g.hasOwnProperty("ngMax")),r=!c.attr("aria-valuenow");q&&g.$observe("min",function(a){c.attr("aria-valuemin",a)});p&&g.$observe("max",function(a){c.attr("aria-valuemax",a)});r&&d.$watch(m,function(a){c.attr("aria-valuenow",a)})}k&&c.attr("tabindex",
0)}!g.hasOwnProperty("ngRequired")&&n.$validators.required&&b("aria-required","ariaRequired",c,!1)&&g.$observe("required",function(){c.attr("aria-required",!!g.required)});b("aria-invalid","ariaInvalid",c,!0)&&d.$watch(function(){return n.$invalid},function(a){c.attr("aria-invalid",!!a)})}}}}}]).directive("ngDisabled",["$aria",function(a){return a.$$watchExpr("ngDisabled","aria-disabled",d,!1)}]).directive("ngMessages",function(){return{restrict:"A",require:"?ngMessages",link:function(a,b,d,h){b.attr("aria-live")||
b.attr("aria-live","assertive")}}}).directive("ngClick",["$aria","$parse",function(a,b){return{restrict:"A",compile:function(f,l){var k=b(l.ngClick);return function(b,e,f){if(!h(e,d)&&(a.config("bindRoleForClick")&&!e.attr("role")&&e.attr("role","button"),a.config("tabindex")&&!e.attr("tabindex")&&e.attr("tabindex",0),a.config("bindKeydown")&&!f.ngKeydown&&!f.ngKeypress&&!f.ngKeyup))e.on("keydown",function(a){function c(){k(b,{$event:a})}var d=a.which||a.keyCode;32!==d&&13!==d||b.$apply(c)})}}}}]).directive("ngDblclick",
["$aria",function(a){return function(b,f,l){!a.config("tabindex")||f.attr("tabindex")||h(f,d)||f.attr("tabindex",0)}}])})(window,window.angular);

//javascript/closure/i18n/bidi.js
// Copyright 2007 The Closure Library Authors. All Rights Reserved.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//      http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS-IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

/**
 * @fileoverview Utility functions for supporting Bidi issues.
 */


/**
 * Namespace for bidi supporting functions.
 */
goog.provide('goog.i18n.bidi');
goog.provide('goog.i18n.bidi.Dir');
goog.provide('goog.i18n.bidi.DirectionalString');
goog.provide('goog.i18n.bidi.Format');


/**
 * @define {boolean} FORCE_RTL forces the {@link goog.i18n.bidi.IS_RTL} constant
 * to say that the current locale is a RTL locale.  This should only be used
 * if you want to override the default behavior for deciding whether the
 * current locale is RTL or not.
 *
 * {@see goog.i18n.bidi.IS_RTL}
 */
goog.i18n.bidi.FORCE_RTL = goog.define('goog.i18n.bidi.FORCE_RTL', false);


/**
 * Constant that defines whether or not the current locale is a RTL locale.
 * If {@link goog.i18n.bidi.FORCE_RTL} is not true, this constant will default
 * to check that {@link goog.LOCALE} is one of a few major RTL locales.
 *
 * <p>This is designed to be a maximally efficient compile-time constant. For
 * example, for the default goog.LOCALE, compiling
 * "if (goog.i18n.bidi.IS_RTL) alert('rtl') else {}" should produce no code. It
 * is this design consideration that limits the implementation to only
 * supporting a few major RTL locales, as opposed to the broader repertoire of
 * something like goog.i18n.bidi.isRtlLanguage.
 *
 * <p>Since this constant refers to the directionality of the locale, it is up
 * to the caller to determine if this constant should also be used for the
 * direction of the UI.
 *
 * {@see goog.LOCALE}
 *
 * @type {boolean}
 *
 * TODO(aharon): write a test that checks that this is a compile-time constant.
 */
// LINT.IfChange
goog.i18n.bidi.IS_RTL =
    goog.i18n.bidi.FORCE_RTL ||
    ((goog.LOCALE.substring(0, 2).toLowerCase() == 'ar' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'fa' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'he' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'iw' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'ps' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'sd' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'ug' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'ur' ||
      goog.LOCALE.substring(0, 2).toLowerCase() == 'yi') &&
     (goog.LOCALE.length == 2 || goog.LOCALE.substring(2, 3) == '-' ||
      goog.LOCALE.substring(2, 3) == '_')) ||
    (  // Specific to CKB (Central Kurdish)
        goog.LOCALE.length >= 3 &&
        goog.LOCALE.substring(0, 3).toLowerCase() == 'ckb' &&
        (goog.LOCALE.length == 3 || goog.LOCALE.substring(3, 4) == '-' ||
         goog.LOCALE.substring(3, 4) == '_')) ||
    (  // 2 letter language codes with RTL scripts
        goog.LOCALE.length >= 7 &&
        ((goog.LOCALE.substring(2, 3) == '-' ||
          goog.LOCALE.substring(2, 3) == '_') &&
         (goog.LOCALE.substring(3, 7).toLowerCase() == 'adlm' ||
          goog.LOCALE.substring(3, 7).toLowerCase() == 'arab' ||
          goog.LOCALE.substring(3, 7).toLowerCase() == 'hebr' ||
          goog.LOCALE.substring(3, 7).toLowerCase() == 'nkoo' ||
          goog.LOCALE.substring(3, 7).toLowerCase() == 'rohg' ||
          goog.LOCALE.substring(3, 7).toLowerCase() == 'thaa'))) ||
    (  // 3 letter languages codes with RTL scripts
        goog.LOCALE.length >= 8 &&
        ((goog.LOCALE.substring(3, 4) == '-' ||
          goog.LOCALE.substring(3, 4) == '_') &&
         (goog.LOCALE.substring(4, 8).toLowerCase() == 'adlm' ||
          goog.LOCALE.substring(4, 8).toLowerCase() == 'arab' ||
          goog.LOCALE.substring(4, 8).toLowerCase() == 'hebr' ||
          goog.LOCALE.substring(4, 8).toLowerCase() == 'nkoo' ||
          goog.LOCALE.substring(4, 8).toLowerCase() == 'rohg' ||
          goog.LOCALE.substring(4, 8).toLowerCase() == 'thaa')));
//    closure/RtlLocalesTest.java)

// TODO(b/77919903): Add additional scripts and languages that are RTL,
// e.g., mende, samaritan, etc.


/**
 * Unicode formatting characters and directionality string constants.
 * @enum {string}
 */
goog.i18n.bidi.Format = {
  /** Unicode "Left-To-Right Embedding" (LRE) character. */
  LRE: '\u202A',
  /** Unicode "Right-To-Left Embedding" (RLE) character. */
  RLE: '\u202B',
  /** Unicode "Pop Directional Formatting" (PDF) character. */
  PDF: '\u202C',
  /** Unicode "Left-To-Right Mark" (LRM) character. */
  LRM: '\u200E',
  /** Unicode "Right-To-Left Mark" (RLM) character. */
  RLM: '\u200F'
};


/**
 * Directionality enum.
 * @enum {number}
 */
goog.i18n.bidi.Dir = {
  /**
   * Left-to-right.
   */
  LTR: 1,

  /**
   * Right-to-left.
   */
  RTL: -1,

  /**
   * Neither left-to-right nor right-to-left.
   */
  NEUTRAL: 0
};


/**
 * 'right' string constant.
 * @type {string}
 */
goog.i18n.bidi.RIGHT = 'right';


/**
 * 'left' string constant.
 * @type {string}
 */
goog.i18n.bidi.LEFT = 'left';


/**
 * 'left' if locale is RTL, 'right' if not.
 * @type {string}
 */
goog.i18n.bidi.I18N_RIGHT =
    goog.i18n.bidi.IS_RTL ? goog.i18n.bidi.LEFT : goog.i18n.bidi.RIGHT;


/**
 * 'right' if locale is RTL, 'left' if not.
 * @type {string}
 */
goog.i18n.bidi.I18N_LEFT =
    goog.i18n.bidi.IS_RTL ? goog.i18n.bidi.RIGHT : goog.i18n.bidi.LEFT;


/**
 * Convert a directionality given in various formats to a goog.i18n.bidi.Dir
 * constant. Useful for interaction with different standards of directionality
 * representation.
 *
 * @param {goog.i18n.bidi.Dir|number|boolean|null} givenDir Directionality given
 *     in one of the following formats:
 *     1. A goog.i18n.bidi.Dir constant.
 *     2. A number (positive = LTR, negative = RTL, 0 = neutral).
 *     3. A boolean (true = RTL, false = LTR).
 *     4. A null for unknown directionality.
 * @param {boolean=} opt_noNeutral Whether a givenDir of zero or
 *     goog.i18n.bidi.Dir.NEUTRAL should be treated as null, i.e. unknown, in
 *     order to preserve legacy behavior.
 * @return {?goog.i18n.bidi.Dir} A goog.i18n.bidi.Dir constant matching the
 *     given directionality. If given null, returns null (i.e. unknown).
 */
goog.i18n.bidi.toDir = function(givenDir, opt_noNeutral) {
  if (typeof givenDir == 'number') {
    // This includes the non-null goog.i18n.bidi.Dir case.
    return givenDir > 0 ?
        goog.i18n.bidi.Dir.LTR :
        givenDir < 0 ? goog.i18n.bidi.Dir.RTL :
                       opt_noNeutral ? null : goog.i18n.bidi.Dir.NEUTRAL;
  } else if (givenDir == null) {
    return null;
  } else {
    // Must be typeof givenDir == 'boolean'.
    return givenDir ? goog.i18n.bidi.Dir.RTL : goog.i18n.bidi.Dir.LTR;
  }
};


/**
 * A practical pattern to identify strong LTR character in the BMP.
 * This pattern is not theoretically correct according to the Unicode
 * standard. It is simplified for performance and small code size.
 * It also partially supports LTR scripts beyond U+FFFF by including
 * UTF-16 high surrogate values corresponding to mostly L-class code
 * point ranges.
 * However, low surrogate values and private-use regions are not included
 * in this RegEx.
 * @type {string}
 * @private
 */
goog.i18n.bidi.ltrChars_ =
    'A-Za-z\u00C0-\u00D6\u00D8-\u00F6\u00F8-\u02B8\u0300-\u0590\u0900-\u1FFF' +
    '\u200E\u2C00-\uD801\uD804-\uD839\uD83C-\uDBFF' +
    '\uF900-\uFB1C\uFE00-\uFE6F\uFEFD-\uFFFF';

/**
 * A practical pattern to identify strong RTL character. This pattern is not
 * theoretically correct according to the Unicode standard. It is simplified
 * for performance and small code size.
 * It also partially supports RTL scripts beyond U+FFFF by including
 * UTF-16 high surrogate values corresponding to mostly R- or AL-class
 * code point ranges.
 * However, low surrogate values and private-use regions are not included
 * in this RegEx.
 * @type {string}
 * @private
 */
goog.i18n.bidi.rtlChars_ =
    '\u0591-\u06EF\u06FA-\u08FF\u200F\uD802-\uD803\uD83A-\uD83B' +
    '\uFB1D-\uFDFF\uFE70-\uFEFC';

/**
 * Simplified regular expression for an HTML tag (opening or closing) or an HTML
 * escape. We might want to skip over such expressions when estimating the text
 * directionality.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.htmlSkipReg_ = /<[^>]*>|&[^;]+;/g;


/**
 * Returns the input text with spaces instead of HTML tags or HTML escapes, if
 * opt_isStripNeeded is true. Else returns the input as is.
 * Useful for text directionality estimation.
 * Note: the function should not be used in other contexts; it is not 100%
 * correct, but rather a good-enough implementation for directionality
 * estimation purposes.
 * @param {string} str The given string.
 * @param {boolean=} opt_isStripNeeded Whether to perform the stripping.
 *     Default: false (to retain consistency with calling functions).
 * @return {string} The given string cleaned of HTML tags / escapes.
 * @private
 */
goog.i18n.bidi.stripHtmlIfNeeded_ = function(str, opt_isStripNeeded) {
  return opt_isStripNeeded ? str.replace(goog.i18n.bidi.htmlSkipReg_, '') : str;
};


/**
 * Regular expression to check for RTL characters, BMP and high surrogate.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.rtlCharReg_ = new RegExp('[' + goog.i18n.bidi.rtlChars_ + ']');


/**
 * Regular expression to check for LTR characters.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.ltrCharReg_ = new RegExp('[' + goog.i18n.bidi.ltrChars_ + ']');


/**
 * Test whether the given string has any RTL characters in it.
 * @param {string} str The given string that need to be tested.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether the string contains RTL characters.
 */
goog.i18n.bidi.hasAnyRtl = function(str, opt_isHtml) {
  return goog.i18n.bidi.rtlCharReg_.test(
      goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml));
};


/**
 * Test whether the given string has any RTL characters in it.
 * @param {string} str The given string that need to be tested.
 * @return {boolean} Whether the string contains RTL characters.
 * @deprecated Use hasAnyRtl.
 */
goog.i18n.bidi.hasRtlChar = goog.i18n.bidi.hasAnyRtl;


/**
 * Test whether the given string has any LTR characters in it.
 * @param {string} str The given string that need to be tested.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether the string contains LTR characters.
 */
goog.i18n.bidi.hasAnyLtr = function(str, opt_isHtml) {
  return goog.i18n.bidi.ltrCharReg_.test(
      goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml));
};


/**
 * Regular expression pattern to check if the first character in the string
 * is LTR.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.ltrRe_ = new RegExp('^[' + goog.i18n.bidi.ltrChars_ + ']');


/**
 * Regular expression pattern to check if the first character in the string
 * is RTL.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.rtlRe_ = new RegExp('^[' + goog.i18n.bidi.rtlChars_ + ']');


/**
 * Check if the first character in the string is RTL or not.
 * @param {string} str The given string that need to be tested.
 * @return {boolean} Whether the first character in str is an RTL char.
 */
goog.i18n.bidi.isRtlChar = function(str) {
  return goog.i18n.bidi.rtlRe_.test(str);
};


/**
 * Check if the first character in the string is LTR or not.
 * @param {string} str The given string that need to be tested.
 * @return {boolean} Whether the first character in str is an LTR char.
 */
goog.i18n.bidi.isLtrChar = function(str) {
  return goog.i18n.bidi.ltrRe_.test(str);
};


/**
 * Check if the first character in the string is neutral or not.
 * @param {string} str The given string that need to be tested.
 * @return {boolean} Whether the first character in str is a neutral char.
 */
goog.i18n.bidi.isNeutralChar = function(str) {
  return !goog.i18n.bidi.isLtrChar(str) && !goog.i18n.bidi.isRtlChar(str);
};


/**
 * Regular expressions to check if a piece of text is of LTR directionality
 * on first character with strong directionality.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.ltrDirCheckRe_ = new RegExp(
    '^[^' + goog.i18n.bidi.rtlChars_ + ']*[' + goog.i18n.bidi.ltrChars_ + ']');


/**
 * Regular expressions to check if a piece of text is of RTL directionality
 * on first character with strong directionality.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.rtlDirCheckRe_ = new RegExp(
    '^[^' + goog.i18n.bidi.ltrChars_ + ']*[' + goog.i18n.bidi.rtlChars_ + ']');


/**
 * Check whether the first strongly directional character (if any) is RTL.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether RTL directionality is detected using the first
 *     strongly-directional character method.
 */
goog.i18n.bidi.startsWithRtl = function(str, opt_isHtml) {
  return goog.i18n.bidi.rtlDirCheckRe_.test(
      goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml));
};


/**
 * Check whether the first strongly directional character (if any) is RTL.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether RTL directionality is detected using the first
 *     strongly-directional character method.
 * @deprecated Use startsWithRtl.
 */
goog.i18n.bidi.isRtlText = goog.i18n.bidi.startsWithRtl;


/**
 * Check whether the first strongly directional character (if any) is LTR.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether LTR directionality is detected using the first
 *     strongly-directional character method.
 */
goog.i18n.bidi.startsWithLtr = function(str, opt_isHtml) {
  return goog.i18n.bidi.ltrDirCheckRe_.test(
      goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml));
};


/**
 * Check whether the first strongly directional character (if any) is LTR.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether LTR directionality is detected using the first
 *     strongly-directional character method.
 * @deprecated Use startsWithLtr.
 */
goog.i18n.bidi.isLtrText = goog.i18n.bidi.startsWithLtr;


/**
 * Regular expression to check if a string looks like something that must
 * always be LTR even in RTL text, e.g. a URL. When estimating the
 * directionality of text containing these, we treat these as weakly LTR,
 * like numbers.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.isRequiredLtrRe_ = /^http:\/\/.*/;


/**
 * Check whether the input string either contains no strongly directional
 * characters or looks like a url.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether neutral directionality is detected.
 */
goog.i18n.bidi.isNeutralText = function(str, opt_isHtml) {
  str = goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml);
  return goog.i18n.bidi.isRequiredLtrRe_.test(str) ||
      !goog.i18n.bidi.hasAnyLtr(str) && !goog.i18n.bidi.hasAnyRtl(str);
};


/**
 * Regular expressions to check if the last strongly-directional character in a
 * piece of text is LTR.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.ltrExitDirCheckRe_ = new RegExp(
    '[' + goog.i18n.bidi.ltrChars_ + ']' +
    '[^' + goog.i18n.bidi.rtlChars_ + ']*$');


/**
 * Regular expressions to check if the last strongly-directional character in a
 * piece of text is RTL.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.rtlExitDirCheckRe_ = new RegExp(
    '[' + goog.i18n.bidi.rtlChars_ + ']' +
    '[^' + goog.i18n.bidi.ltrChars_ + ']*$');


/**
 * Check if the exit directionality a piece of text is LTR, i.e. if the last
 * strongly-directional character in the string is LTR.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether LTR exit directionality was detected.
 */
goog.i18n.bidi.endsWithLtr = function(str, opt_isHtml) {
  return goog.i18n.bidi.ltrExitDirCheckRe_.test(
      goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml));
};


/**
 * Check if the exit directionality a piece of text is LTR, i.e. if the last
 * strongly-directional character in the string is LTR.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether LTR exit directionality was detected.
 * @deprecated Use endsWithLtr.
 */
goog.i18n.bidi.isLtrExitText = goog.i18n.bidi.endsWithLtr;


/**
 * Check if the exit directionality a piece of text is RTL, i.e. if the last
 * strongly-directional character in the string is RTL.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether RTL exit directionality was detected.
 */
goog.i18n.bidi.endsWithRtl = function(str, opt_isHtml) {
  return goog.i18n.bidi.rtlExitDirCheckRe_.test(
      goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml));
};


/**
 * Check if the exit directionality a piece of text is RTL, i.e. if the last
 * strongly-directional character in the string is RTL.
 * @param {string} str String being checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether RTL exit directionality was detected.
 * @deprecated Use endsWithRtl.
 */
goog.i18n.bidi.isRtlExitText = goog.i18n.bidi.endsWithRtl;


/**
 * A regular expression for matching right-to-left language codes.
 * See {@link #isRtlLanguage} for the design.
 * Note that not all RTL scripts are included.
 * @type {!RegExp}
 * @private
 */
goog.i18n.bidi.rtlLocalesRe_ = new RegExp(
    '^(ar|ckb|dv|he|iw|fa|nqo|ps|sd|ug|ur|yi|' +
        '.*[-_](Adlm|Arab|Hebr|Nkoo|Rohg|Thaa))' +
        '(?!.*[-_](Latn|Cyrl)($|-|_))($|-|_)',
    'i');


/**
 * Check if a BCP 47 / III language code indicates an RTL language, i.e. either:
 * - a language code explicitly specifying one of the right-to-left scripts,
 *   e.g. "az-Arab", or<p>
 * - a language code specifying one of the languages normally written in a
 *   right-to-left script, e.g. "fa" (Farsi), except ones explicitly specifying
 *   Latin or Cyrillic script (which are the usual LTR alternatives).<p>
 * The list of right-to-left scripts appears in the 100-199 range in
 * http://www.unicode.org/iso15924/iso15924-num.html, of which Arabic and
 * Hebrew are by far the most widely used. We also recognize Thaana, and N'Ko,
 * which also have significant modern usage. Adlam and Rohingya
 * scripts are now included since they can be expected to be used in the
 * future. The rest (Syriac, Samaritan, Mandaic, etc.) seem to have extremely
 * limited or no modern usage and are not recognized to save on code size. The
 * languages usually written in a right-to-left script are taken as those with
 * Suppress-Script: Hebr|Arab|Thaa|Nkoo|Adlm|Rohg in
 * http://www.iana.org/assignments/language-subtag-registry,
 * as well as Central (or Sorani) Kurdish (ckb), Sindhi (sd) and Uyghur (ug).
 * Other subtags of the language code, e.g. regions like EG (Egypt), are
 * ignored.
 * @param {string} lang BCP 47 (a.k.a III) language code.
 * @return {boolean} Whether the language code is an RTL language.
 */
goog.i18n.bidi.isRtlLanguage = function(lang) {
  return goog.i18n.bidi.rtlLocalesRe_.test(lang);
};


/**
 * Regular expression for bracket guard replacement in text.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.bracketGuardTextRe_ =
    /(\(.*?\)+)|(\[.*?\]+)|(\{.*?\}+)|(<.*?>+)/g;


/**
 * Apply bracket guard using LRM and RLM. This is to address the problem of
 * messy bracket display frequently happens in RTL layout.
 * This function works for plain text, not for HTML. In HTML, the opening
 * bracket might be in a different context than the closing bracket (such as
 * an attribute value).
 * @param {string} s The string that need to be processed.
 * @param {boolean=} opt_isRtlContext specifies default direction (usually
 *     direction of the UI).
 * @return {string} The processed string, with all bracket guarded.
 */
goog.i18n.bidi.guardBracketInText = function(s, opt_isRtlContext) {
  const useRtl = opt_isRtlContext === undefined ? goog.i18n.bidi.hasAnyRtl(s) :
                                                  opt_isRtlContext;
  const mark = useRtl ? goog.i18n.bidi.Format.RLM : goog.i18n.bidi.Format.LRM;
  return s.replace(goog.i18n.bidi.bracketGuardTextRe_, mark + '$&' + mark);
};


/**
 * Enforce the html snippet in RTL directionality regardless of overall context.
 * If the html piece was enclosed by tag, dir will be applied to existing
 * tag, otherwise a span tag will be added as wrapper. For this reason, if
 * html snippet starts with a tag, this tag must enclose the whole piece. If
 * the tag already has a dir specified, this new one will override existing
 * one in behavior (tested on FF and IE).
 * @param {string} html The string that need to be processed.
 * @return {string} The processed string, with directionality enforced to RTL.
 */
goog.i18n.bidi.enforceRtlInHtml = function(html) {
  if (html.charAt(0) == '<') {
    return html.replace(/<\w+/, '$& dir=rtl');
  }
  // '\n' is important for FF so that it won't incorrectly merge span groups
  return '\n<span dir=rtl>' + html + '</span>';
};


/**
 * Enforce RTL on both end of the given text piece using unicode BiDi formatting
 * characters RLE and PDF.
 * @param {string} text The piece of text that need to be wrapped.
 * @return {string} The wrapped string after process.
 */
goog.i18n.bidi.enforceRtlInText = function(text) {
  return goog.i18n.bidi.Format.RLE + text + goog.i18n.bidi.Format.PDF;
};


/**
 * Enforce the html snippet in RTL directionality regardless or overall context.
 * If the html piece was enclosed by tag, dir will be applied to existing
 * tag, otherwise a span tag will be added as wrapper. For this reason, if
 * html snippet starts with a tag, this tag must enclose the whole piece. If
 * the tag already has a dir specified, this new one will override existing
 * one in behavior (tested on FF and IE).
 * @param {string} html The string that need to be processed.
 * @return {string} The processed string, with directionality enforced to RTL.
 */
goog.i18n.bidi.enforceLtrInHtml = function(html) {
  if (html.charAt(0) == '<') {
    return html.replace(/<\w+/, '$& dir=ltr');
  }
  // '\n' is important for FF so that it won't incorrectly merge span groups
  return '\n<span dir=ltr>' + html + '</span>';
};


/**
 * Enforce LTR on both end of the given text piece using unicode BiDi formatting
 * characters LRE and PDF.
 * @param {string} text The piece of text that need to be wrapped.
 * @return {string} The wrapped string after process.
 */
goog.i18n.bidi.enforceLtrInText = function(text) {
  return goog.i18n.bidi.Format.LRE + text + goog.i18n.bidi.Format.PDF;
};


/**
 * Regular expression to find dimensions such as "padding: .3 0.4ex 5px 6;"
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.dimensionsRe_ =
    /:\s*([.\d][.\w]*)\s+([.\d][.\w]*)\s+([.\d][.\w]*)\s+([.\d][.\w]*)/g;


/**
 * Regular expression for left.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.leftRe_ = /left/gi;


/**
 * Regular expression for right.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.rightRe_ = /right/gi;


/**
 * Placeholder regular expression for swapping.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.tempRe_ = /%%%%/g;


/**
 * Swap location parameters and 'left'/'right' in CSS specification. The
 * processed string will be suited for RTL layout. Though this function can
 * cover most cases, there are always exceptions. It is suggested to put
 * those exceptions in separate group of CSS string.
 * @param {string} cssStr CSS spefication string.
 * @return {string} Processed CSS specification string.
 */
goog.i18n.bidi.mirrorCSS = function(cssStr) {
  return cssStr
      .
      // reverse dimensions
      replace(goog.i18n.bidi.dimensionsRe_, ':$1 $4 $3 $2')
      .replace(goog.i18n.bidi.leftRe_, '%%%%')
      .  // swap left and right
      replace(goog.i18n.bidi.rightRe_, goog.i18n.bidi.LEFT)
      .replace(goog.i18n.bidi.tempRe_, goog.i18n.bidi.RIGHT);
};


/**
 * Regular expression for hebrew double quote substitution, finding quote
 * directly after hebrew characters.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.doubleQuoteSubstituteRe_ = /([\u0591-\u05f2])"/g;


/**
 * Regular expression for hebrew single quote substitution, finding quote
 * directly after hebrew characters.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.singleQuoteSubstituteRe_ = /([\u0591-\u05f2])'/g;


/**
 * Replace the double and single quote directly after a Hebrew character with
 * GERESH and GERSHAYIM. In such case, most likely that's user intention.
 * @param {string} str String that need to be processed.
 * @return {string} Processed string with double/single quote replaced.
 */
goog.i18n.bidi.normalizeHebrewQuote = function(str) {
  return str.replace(goog.i18n.bidi.doubleQuoteSubstituteRe_, '$1\u05f4')
      .replace(goog.i18n.bidi.singleQuoteSubstituteRe_, '$1\u05f3');
};


/**
 * Regular expression to split a string into "words" for directionality
 * estimation based on relative word counts.
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.wordSeparatorRe_ = /\s+/;


/**
 * Regular expression to check if a string contains any numerals. Used to
 * differentiate between completely neutral strings and those containing
 * numbers, which are weakly LTR.
 *
 * Native Arabic digits (\u0660 - \u0669) are not included because although they
 * do flow left-to-right inside a number, this is the case even if the  overall
 * directionality is RTL, and a mathematical expression using these digits is
 * supposed to flow right-to-left overall, including unary plus and minus
 * appearing to the right of a number, and this does depend on the overall
 * directionality being RTL. The digits used in Farsi (\u06F0 - \u06F9), on the
 * other hand, are included, since Farsi math (including unary plus and minus)
 * does flow left-to-right.
 * TODO: Consider other systems of digits, e.g., Adlam.
 *
 * @type {RegExp}
 * @private
 */
goog.i18n.bidi.hasNumeralsRe_ = /[\d\u06f0-\u06f9]/;


/**
 * This constant controls threshold of RTL directionality.
 * @type {number}
 * @private
 */
goog.i18n.bidi.rtlDetectionThreshold_ = 0.40;


/**
 * Estimates the directionality of a string based on relative word counts.
 * If the number of RTL words is above a certain percentage of the total number
 * of strongly directional words, returns RTL.
 * Otherwise, if any words are strongly or weakly LTR, returns LTR.
 * Otherwise, returns UNKNOWN, which is used to mean "neutral".
 * Numbers are counted as weakly LTR.
 * @param {string} str The string to be checked.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {goog.i18n.bidi.Dir} Estimated overall directionality of `str`.
 */
goog.i18n.bidi.estimateDirection = function(str, opt_isHtml) {
  let rtlCount = 0;
  let totalCount = 0;
  let hasWeaklyLtr = false;
  const tokens = goog.i18n.bidi.stripHtmlIfNeeded_(str, opt_isHtml)
                     .split(goog.i18n.bidi.wordSeparatorRe_);
  for (let i = 0; i < tokens.length; i++) {
    const token = tokens[i];
    if (goog.i18n.bidi.startsWithRtl(token)) {
      rtlCount++;
      totalCount++;
    } else if (goog.i18n.bidi.isRequiredLtrRe_.test(token)) {
      hasWeaklyLtr = true;
    } else if (goog.i18n.bidi.hasAnyLtr(token)) {
      totalCount++;
    } else if (goog.i18n.bidi.hasNumeralsRe_.test(token)) {
      hasWeaklyLtr = true;
    }
  }

  return totalCount == 0 ?
      (hasWeaklyLtr ? goog.i18n.bidi.Dir.LTR : goog.i18n.bidi.Dir.NEUTRAL) :
      (rtlCount / totalCount > goog.i18n.bidi.rtlDetectionThreshold_ ?
           goog.i18n.bidi.Dir.RTL :
           goog.i18n.bidi.Dir.LTR);
};


/**
 * Check the directionality of a piece of text, return true if the piece of
 * text should be laid out in RTL direction.
 * @param {string} str The piece of text that need to be detected.
 * @param {boolean=} opt_isHtml Whether str is HTML / HTML-escaped.
 *     Default: false.
 * @return {boolean} Whether this piece of text should be laid out in RTL.
 */
goog.i18n.bidi.detectRtlDirectionality = function(str, opt_isHtml) {
  return goog.i18n.bidi.estimateDirection(str, opt_isHtml) ==
      goog.i18n.bidi.Dir.RTL;
};


/**
 * Sets text input element's directionality and text alignment based on a
 * given directionality. Does nothing if the given directionality is unknown or
 * neutral.
 * @param {Element} element Input field element to set directionality to.
 * @param {goog.i18n.bidi.Dir|number|boolean|null} dir Desired directionality,
 *     given in one of the following formats:
 *     1. A goog.i18n.bidi.Dir constant.
 *     2. A number (positive = LRT, negative = RTL, 0 = neutral).
 *     3. A boolean (true = RTL, false = LTR).
 *     4. A null for unknown directionality.
 */
goog.i18n.bidi.setElementDirAndAlign = function(element, dir) {
  if (element) {
    const htmlElement = /** @type {!HTMLElement} */ (element);
    dir = goog.i18n.bidi.toDir(dir);
    if (dir) {
      htmlElement.style.textAlign = dir == goog.i18n.bidi.Dir.RTL ?
          goog.i18n.bidi.RIGHT :
          goog.i18n.bidi.LEFT;
      htmlElement.dir = dir == goog.i18n.bidi.Dir.RTL ? 'rtl' : 'ltr';
    }
  }
};


/**
 * Sets element dir based on estimated directionality of the given text.
 * @param {!Element} element
 * @param {string} text
 */
goog.i18n.bidi.setElementDirByTextDirectionality = function(element, text) {
  const htmlElement = /** @type {!HTMLElement} */ (element);
  switch (goog.i18n.bidi.estimateDirection(text)) {
    case (goog.i18n.bidi.Dir.LTR):
      htmlElement.dir = 'ltr';
      break;
    case (goog.i18n.bidi.Dir.RTL):
      htmlElement.dir = 'rtl';
      break;
    default:
      // Default for no direction, inherit from document.
      htmlElement.removeAttribute('dir');
  }
};



/**
 * Strings that have an (optional) known direction.
 *
 * Implementations of this interface are string-like objects that carry an
 * attached direction, if known.
 * @interface
 */
goog.i18n.bidi.DirectionalString = function() {};


/**
 * Interface marker of the DirectionalString interface.
 *
 * This property can be used to determine at runtime whether or not an object
 * implements this interface.  All implementations of this interface set this
 * property to `true`.
 * @type {boolean}
 */
goog.i18n.bidi.DirectionalString.prototype
    .implementsGoogI18nBidiDirectionalString;


/**
 * Retrieves this object's known direction (if any).
 * @return {?goog.i18n.bidi.Dir} The known direction. Null if unknown.
 */
goog.i18n.bidi.DirectionalString.prototype.getDirection;

//third_party/javascript/closure/debug/error.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Provides a base class for custom Error objects such that the
 * stack is correctly maintained.
 *
 * You should never need to throw goog.debug.Error(msg) directly, Error(msg) is
 * sufficient.
 */

goog.provide('goog.debug.Error');



/**
 * Base class for custom error objects.
 * @param {*=} opt_msg The message associated with the error.
 * @constructor
 * @extends {Error}
 */
goog.debug.Error = function(opt_msg) {

  // Attempt to ensure there is a stack trace.
  if (Error.captureStackTrace) {
    Error.captureStackTrace(this, goog.debug.Error);
  } else {
    const stack = new Error().stack;
    if (stack) {
      /** @override */
      this.stack = stack;
    }
  }

  if (opt_msg) {
    /** @override */
    this.message = String(opt_msg);
  }

  /**
   * Whether to report this error to the server. Setting this to false will
   * cause the error reporter to not report the error back to the server,
   * which can be useful if the client knows that the error has already been
   * logged on the server.
   * @type {boolean}
   */
  this.reportErrorToServer = true;
};
goog.inherits(goog.debug.Error, Error);


/** @override */
goog.debug.Error.prototype.name = 'CustomError';

//third_party/javascript/closure/dom/nodetype.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Definition of goog.dom.NodeType.
 */

goog.provide('goog.dom.NodeType');


/**
 * Constants for the nodeType attribute in the Node interface.
 *
 * These constants match those specified in the Node interface. These are
 * usually present on the Node object in recent browsers, but not in older
 * browsers (specifically, early IEs) and thus are given here.
 *
 * In some browsers (early IEs), these are not defined on the Node object,
 * so they are provided here.
 *
 * See http://www.w3.org/TR/DOM-Level-2-Core/core.html#ID-1950641247
 * @enum {number}
 */
goog.dom.NodeType = {
  ELEMENT: 1,
  ATTRIBUTE: 2,
  TEXT: 3,
  CDATA_SECTION: 4,
  ENTITY_REFERENCE: 5,
  ENTITY: 6,
  PROCESSING_INSTRUCTION: 7,
  COMMENT: 8,
  DOCUMENT: 9,
  DOCUMENT_TYPE: 10,
  DOCUMENT_FRAGMENT: 11,
  NOTATION: 12
};

//third_party/javascript/closure/asserts/asserts.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Utilities to check the preconditions, postconditions and
 * invariants runtime.
 *
 * Methods in this package are given special treatment by the compiler
 * for type-inference. For example, <code>goog.asserts.assert(foo)</code>
 * will make the compiler treat <code>foo</code> as non-nullable. Similarly,
 * <code>goog.asserts.assertNumber(foo)</code> informs the compiler about the
 * type of <code>foo</code>. Where applicable, such assertions are preferable to
 * casts by jsdoc with <code>@type</code>.
 *
 * The compiler has an option to disable asserts. So code like:
 * <code>
 * var x = goog.asserts.assert(foo());
 * goog.asserts.assert(bar());
 * </code>
 * will be transformed into:
 * <code>
 * var x = foo();
 * </code>
 * The compiler will leave in foo() (because its return value is used),
 * but it will remove bar() because it assumes it does not have side-effects.
 *
 * Additionally, note the compiler will consider the type to be "tightened" for
 * all statements <em>after</em> the assertion. For example:
 * <code>
 * const /** ?Object &#ast;/ value = foo();
 * goog.asserts.assert(value);
 * // "value" is of type {!Object} at this point.
 * </code>
 */

goog.provide('goog.asserts');
goog.provide('goog.asserts.AssertionError');

goog.require('goog.debug.Error');
goog.require('goog.dom.NodeType');


/**
 * @define {boolean} Whether to strip out asserts or to leave them in.
 */
goog.asserts.ENABLE_ASSERTS =
    goog.define('goog.asserts.ENABLE_ASSERTS', goog.DEBUG);



/**
 * Error object for failed assertions.
 * @param {string} messagePattern The pattern that was used to form message.
 * @param {!Array<*>} messageArgs The items to substitute into the pattern.
 * @constructor
 * @extends {goog.debug.Error}
 * @final
 */
goog.asserts.AssertionError = function(messagePattern, messageArgs) {
  goog.debug.Error.call(this, goog.asserts.subs_(messagePattern, messageArgs));

  /**
   * The message pattern used to format the error message. Error handlers can
   * use this to uniquely identify the assertion.
   * @type {string}
   */
  this.messagePattern = messagePattern;
};
goog.inherits(goog.asserts.AssertionError, goog.debug.Error);


/** @override */
goog.asserts.AssertionError.prototype.name = 'AssertionError';


/**
 * The default error handler.
 * @param {!goog.asserts.AssertionError} e The exception to be handled.
 */
goog.asserts.DEFAULT_ERROR_HANDLER = function(e) {
  throw e;
};


/**
 * The handler responsible for throwing or logging assertion errors.
 * @private {function(!goog.asserts.AssertionError)}
 */
goog.asserts.errorHandler_ = goog.asserts.DEFAULT_ERROR_HANDLER;


/**
 * Does simple python-style string substitution.
 * subs("foo%s hot%s", "bar", "dog") becomes "foobar hotdog".
 * @param {string} pattern The string containing the pattern.
 * @param {!Array<*>} subs The items to substitute into the pattern.
 * @return {string} A copy of `str` in which each occurrence of
 *     {@code %s} has been replaced an argument from `var_args`.
 * @private
 */
goog.asserts.subs_ = function(pattern, subs) {
  var splitParts = pattern.split('%s');
  var returnString = '';

  // Replace up to the last split part. We are inserting in the
  // positions between split parts.
  var subLast = splitParts.length - 1;
  for (var i = 0; i < subLast; i++) {
    // keep unsupplied as '%s'
    var sub = (i < subs.length) ? subs[i] : '%s';
    returnString += splitParts[i] + sub;
  }
  return returnString + splitParts[subLast];
};


/**
 * Throws an exception with the given message and "Assertion failed" prefixed
 * onto it.
 * @param {string} defaultMessage The message to use if givenMessage is empty.
 * @param {Array<*>} defaultArgs The substitution arguments for defaultMessage.
 * @param {string|undefined} givenMessage Message supplied by the caller.
 * @param {Array<*>} givenArgs The substitution arguments for givenMessage.
 * @throws {goog.asserts.AssertionError} When the value is not a number.
 * @private
 */
goog.asserts.doAssertFailure_ = function(
    defaultMessage, defaultArgs, givenMessage, givenArgs) {
  var message = 'Assertion failed';
  if (givenMessage) {
    message += ': ' + givenMessage;
    var args = givenArgs;
  } else if (defaultMessage) {
    message += ': ' + defaultMessage;
    args = defaultArgs;
  }
  // The '' + works around an Opera 10 bug in the unit tests. Without it,
  // a stack trace is added to var message above. With this, a stack trace is
  // not added until this line (it causes the extra garbage to be added after
  // the assertion message instead of in the middle of it).
  var e = new goog.asserts.AssertionError('' + message, args || []);
  goog.asserts.errorHandler_(e);
};


/**
 * Sets a custom error handler that can be used to customize the behavior of
 * assertion failures, for example by turning all assertion failures into log
 * messages.
 * @param {function(!goog.asserts.AssertionError)} errorHandler
 */
goog.asserts.setErrorHandler = function(errorHandler) {
  if (goog.asserts.ENABLE_ASSERTS) {
    goog.asserts.errorHandler_ = errorHandler;
  }
};


/**
 * Checks if the condition evaluates to true if goog.asserts.ENABLE_ASSERTS is
 * true.
 * @template T
 * @param {T} condition The condition to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {T} The value of the condition.
 * @throws {goog.asserts.AssertionError} When the condition evaluates to false.
 * @closurePrimitive {asserts.truthy}
 */
goog.asserts.assert = function(condition, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && !condition) {
    goog.asserts.doAssertFailure_(
        '', null, opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return condition;
};


/**
 * Checks if `value` is `null` or `undefined` if goog.asserts.ENABLE_ASSERTS is
 * true.
 *
 * @param {T} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {R} `value` with its type narrowed to exclude `null` and `undefined`.
 *
 * @template T
 * @template R :=
 *     mapunion(T, (V) =>
 *         cond(eq(V, 'null'),
 *             none(),
 *             cond(eq(V, 'undefined'),
 *                 none(),
 *                 V)))
 *  =:
 *
 * @throws {!goog.asserts.AssertionError} When `value` is `null` or `undefined`.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertExists = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && value == null) {
    goog.asserts.doAssertFailure_(
        'Expected to exist: %s.', [value], opt_message,
        Array.prototype.slice.call(arguments, 2));
  }
  return value;
};


/**
 * Fails if goog.asserts.ENABLE_ASSERTS is true. This function is useful in case
 * when we want to add a check in the unreachable area like switch-case
 * statement:
 *
 * <pre>
 *  switch(type) {
 *    case FOO: doSomething(); break;
 *    case BAR: doSomethingElse(); break;
 *    default: goog.asserts.fail('Unrecognized type: ' + type);
 *      // We have only 2 types - "default:" section is unreachable code.
 *  }
 * </pre>
 *
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @throws {goog.asserts.AssertionError} Failure.
 * @closurePrimitive {asserts.fail}
 */
goog.asserts.fail = function(opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS) {
    goog.asserts.errorHandler_(new goog.asserts.AssertionError(
        'Failure' + (opt_message ? ': ' + opt_message : ''),
        Array.prototype.slice.call(arguments, 1)));
  }
};


/**
 * Checks if the value is a number if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {number} The value, guaranteed to be a number when asserts enabled.
 * @throws {goog.asserts.AssertionError} When the value is not a number.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertNumber = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && typeof value !== 'number') {
    goog.asserts.doAssertFailure_(
        'Expected number but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {number} */ (value);
};


/**
 * Checks if the value is a string if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {string} The value, guaranteed to be a string when asserts enabled.
 * @throws {goog.asserts.AssertionError} When the value is not a string.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertString = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && typeof value !== 'string') {
    goog.asserts.doAssertFailure_(
        'Expected string but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {string} */ (value);
};


/**
 * Checks if the value is a function if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {!Function} The value, guaranteed to be a function when asserts
 *     enabled.
 * @throws {goog.asserts.AssertionError} When the value is not a function.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertFunction = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && !goog.isFunction(value)) {
    goog.asserts.doAssertFailure_(
        'Expected function but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {!Function} */ (value);
};


/**
 * Checks if the value is an Object if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {!Object} The value, guaranteed to be a non-null object.
 * @throws {goog.asserts.AssertionError} When the value is not an object.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertObject = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && !goog.isObject(value)) {
    goog.asserts.doAssertFailure_(
        'Expected object but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {!Object} */ (value);
};


/**
 * Checks if the value is an Array if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {!Array<?>} The value, guaranteed to be a non-null array.
 * @throws {goog.asserts.AssertionError} When the value is not an array.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertArray = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && !Array.isArray(value)) {
    goog.asserts.doAssertFailure_(
        'Expected array but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {!Array<?>} */ (value);
};


/**
 * Checks if the value is a boolean if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {boolean} The value, guaranteed to be a boolean when asserts are
 *     enabled.
 * @throws {goog.asserts.AssertionError} When the value is not a boolean.
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertBoolean = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && typeof value !== 'boolean') {
    goog.asserts.doAssertFailure_(
        'Expected boolean but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {boolean} */ (value);
};


/**
 * Checks if the value is a DOM Element if goog.asserts.ENABLE_ASSERTS is true.
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @return {!Element} The value, likely to be a DOM Element when asserts are
 *     enabled.
 * @throws {goog.asserts.AssertionError} When the value is not an Element.
 * @closurePrimitive {asserts.matchesReturn}
 * @deprecated Use goog.asserts.dom.assertIsElement instead.
 */
goog.asserts.assertElement = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS &&
      (!goog.isObject(value) ||
       /** @type {!Node} */ (value).nodeType != goog.dom.NodeType.ELEMENT)) {
    goog.asserts.doAssertFailure_(
        'Expected Element but got %s: %s.', [goog.typeOf(value), value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {!Element} */ (value);
};


/**
 * Checks if the value is an instance of the user-defined type if
 * goog.asserts.ENABLE_ASSERTS is true.
 *
 * The compiler may tighten the type returned by this function.
 *
 * Do not use this to ensure a value is an HTMLElement or a subclass! Cross-
 * document DOM inherits from separate - though identical - browser classes, and
 * such a check will unexpectedly fail. Please use the methods in
 * goog.asserts.dom for these purposes.
 *
 * @param {?} value The value to check.
 * @param {function(new: T, ...)} type A user-defined constructor.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @throws {goog.asserts.AssertionError} When the value is not an instance of
 *     type.
 * @return {T}
 * @template T
 * @closurePrimitive {asserts.matchesReturn}
 */
goog.asserts.assertInstanceof = function(value, type, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS && !(value instanceof type)) {
    goog.asserts.doAssertFailure_(
        'Expected instanceof %s but got %s.',
        [goog.asserts.getType_(type), goog.asserts.getType_(value)],
        opt_message, Array.prototype.slice.call(arguments, 3));
  }
  return value;
};


/**
 * Checks whether the value is a finite number, if goog.asserts.ENABLE_ASSERTS
 * is true.
 *
 * @param {*} value The value to check.
 * @param {string=} opt_message Error message in case of failure.
 * @param {...*} var_args The items to substitute into the failure message.
 * @throws {goog.asserts.AssertionError} When the value is not a number, or is
 *     a non-finite number such as NaN, Infinity or -Infinity.
 * @return {number} The value initially passed in.
 */
goog.asserts.assertFinite = function(value, opt_message, var_args) {
  if (goog.asserts.ENABLE_ASSERTS &&
      (typeof value != 'number' || !isFinite(value))) {
    goog.asserts.doAssertFailure_(
        'Expected %s to be a finite number but it is not.', [value],
        opt_message, Array.prototype.slice.call(arguments, 2));
  }
  return /** @type {number} */ (value);
};

/**
 * Checks that no enumerable keys are present in Object.prototype. Such keys
 * would break most code that use {@code for (var ... in ...)} loops.
 */
goog.asserts.assertObjectPrototypeIsIntact = function() {
  for (var key in Object.prototype) {
    goog.asserts.fail(key + ' should not be enumerable in Object.prototype.');
  }
};


/**
 * Returns the type of a value. If a constructor is passed, and a suitable
 * string cannot be found, 'unknown type name' will be returned.
 * @param {*} value A constructor, object, or primitive.
 * @return {string} The best display name for the value, or 'unknown type name'.
 * @private
 */
goog.asserts.getType_ = function(value) {
  if (value instanceof Function) {
    return value.displayName || value.name || 'unknown type name';
  } else if (value instanceof Object) {
    return /** @type {string} */ (value.constructor.displayName) ||
        value.constructor.name || Object.prototype.toString.call(value);
  } else {
    return value === null ? 'null' : typeof value;
  }
};

//third_party/javascript/closure/array/array.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Utilities for manipulating arrays.
 */


goog.provide('goog.array');

goog.require('goog.asserts');


/**
 * @define {boolean} NATIVE_ARRAY_PROTOTYPES indicates whether the code should
 * rely on Array.prototype functions, if available.
 *
 * The Array.prototype functions can be defined by external libraries like
 * Prototype and setting this flag to false forces closure to use its own
 * goog.array implementation.
 *
 * If your javascript can be loaded by a third party site and you are wary about
 * relying on the prototype functions, specify
 * "--define goog.NATIVE_ARRAY_PROTOTYPES=false" to the JSCompiler.
 *
 * Setting goog.TRUSTED_SITE to false will automatically set
 * NATIVE_ARRAY_PROTOTYPES to false.
 */
goog.NATIVE_ARRAY_PROTOTYPES =
    goog.define('goog.NATIVE_ARRAY_PROTOTYPES', goog.TRUSTED_SITE);


/**
 * @define {boolean} If true, JSCompiler will use the native implementation of
 * array functions where appropriate (e.g., `Array#filter`) and remove the
 * unused pure JS implementation.
 */
goog.array.ASSUME_NATIVE_FUNCTIONS = goog.define(
    'goog.array.ASSUME_NATIVE_FUNCTIONS', goog.FEATURESET_YEAR > 2012);


/**
 * Returns the last element in an array without removing it.
 * Same as goog.array.last.
 * @param {IArrayLike<T>|string} array The array.
 * @return {T} Last item in array.
 * @template T
 */
goog.array.peek = function(array) {
  return array[array.length - 1];
};


/**
 * Returns the last element in an array without removing it.
 * Same as goog.array.peek.
 * @param {IArrayLike<T>|string} array The array.
 * @return {T} Last item in array.
 * @template T
 */
goog.array.last = goog.array.peek;

// NOTE(arv): Since most of the array functions are generic it allows you to
// pass an array-like object. Strings have a length and are considered array-
// like. However, the 'in' operator does not work on strings so we cannot just
// use the array path even if the browser supports indexing into strings. We
// therefore end up splitting the string.


/**
 * Returns the index of the first element of an array with a specified value, or
 * -1 if the element is not present in the array.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-indexof}
 *
 * @param {IArrayLike<T>|string} arr The array to be searched.
 * @param {T} obj The object for which we are searching.
 * @param {number=} opt_fromIndex The index at which to start the search. If
 *     omitted the search starts at index 0.
 * @return {number} The index of the first matching array element.
 * @template T
 */
goog.array.indexOf = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.indexOf) ?
    function(arr, obj, opt_fromIndex) {
      goog.asserts.assert(arr.length != null);

      return Array.prototype.indexOf.call(arr, obj, opt_fromIndex);
    } :
    function(arr, obj, opt_fromIndex) {
      var fromIndex = opt_fromIndex == null ?
          0 :
          (opt_fromIndex < 0 ? Math.max(0, arr.length + opt_fromIndex) :
                               opt_fromIndex);

      if (typeof arr === 'string') {
        // Array.prototype.indexOf uses === so only strings should be found.
        if (typeof obj !== 'string' || obj.length != 1) {
          return -1;
        }
        return arr.indexOf(obj, fromIndex);
      }

      for (var i = fromIndex; i < arr.length; i++) {
        if (i in arr && arr[i] === obj) return i;
      }
      return -1;
    };


/**
 * Returns the index of the last element of an array with a specified value, or
 * -1 if the element is not present in the array.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-lastindexof}
 *
 * @param {!IArrayLike<T>|string} arr The array to be searched.
 * @param {T} obj The object for which we are searching.
 * @param {?number=} opt_fromIndex The index at which to start the search. If
 *     omitted the search starts at the end of the array.
 * @return {number} The index of the last matching array element.
 * @template T
 */
goog.array.lastIndexOf = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.lastIndexOf) ?
    function(arr, obj, opt_fromIndex) {
      goog.asserts.assert(arr.length != null);

      // Firefox treats undefined and null as 0 in the fromIndex argument which
      // leads it to always return -1
      var fromIndex = opt_fromIndex == null ? arr.length - 1 : opt_fromIndex;
      return Array.prototype.lastIndexOf.call(arr, obj, fromIndex);
    } :
    function(arr, obj, opt_fromIndex) {
      var fromIndex = opt_fromIndex == null ? arr.length - 1 : opt_fromIndex;

      if (fromIndex < 0) {
        fromIndex = Math.max(0, arr.length + fromIndex);
      }

      if (typeof arr === 'string') {
        // Array.prototype.lastIndexOf uses === so only strings should be found.
        if (typeof obj !== 'string' || obj.length != 1) {
          return -1;
        }
        return arr.lastIndexOf(obj, fromIndex);
      }

      for (var i = fromIndex; i >= 0; i--) {
        if (i in arr && arr[i] === obj) return i;
      }
      return -1;
    };


/**
 * Calls a function for each element in an array. Skips holes in the array.
 * See {@link http://tinyurl.com/developer-mozilla-org-array-foreach}
 *
 * @param {IArrayLike<T>|string} arr Array or array like object over
 *     which to iterate.
 * @param {?function(this: S, T, number, ?): ?} f The function to call for every
 *     element. This function takes 3 arguments (the element, the index and the
 *     array). The return value is ignored.
 * @param {S=} opt_obj The object to be used as the value of 'this' within f.
 * @template T,S
 */
goog.array.forEach = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.forEach) ?
    function(arr, f, opt_obj) {
      goog.asserts.assert(arr.length != null);

      Array.prototype.forEach.call(arr, f, opt_obj);
    } :
    function(arr, f, opt_obj) {
      var l = arr.length;  // must be fixed during loop... see docs
      var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
      for (var i = 0; i < l; i++) {
        if (i in arr2) {
          f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr);
        }
      }
    };


/**
 * Calls a function for each element in an array, starting from the last
 * element rather than the first.
 *
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this: S, T, number, ?): ?} f The function to call for every
 *     element. This function
 *     takes 3 arguments (the element, the index and the array). The return
 *     value is ignored.
 * @param {S=} opt_obj The object to be used as the value of 'this'
 *     within f.
 * @template T,S
 */
goog.array.forEachRight = function(arr, f, opt_obj) {
  var l = arr.length;  // must be fixed during loop... see docs
  var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
  for (var i = l - 1; i >= 0; --i) {
    if (i in arr2) {
      f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr);
    }
  }
};


/**
 * Calls a function for each element in an array, and if the function returns
 * true adds the element to a new array.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-filter}
 *
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?):boolean} f The function to call for
 *     every element. This function
 *     takes 3 arguments (the element, the index and the array) and must
 *     return a Boolean. If the return value is true the element is added to the
 *     result array. If it is false the element is not included.
 * @param {S=} opt_obj The object to be used as the value of 'this'
 *     within f.
 * @return {!Array<T>} a new array in which only elements that passed the test
 *     are present.
 * @template T,S
 */
goog.array.filter = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.filter) ?
    function(arr, f, opt_obj) {
      goog.asserts.assert(arr.length != null);

      return Array.prototype.filter.call(arr, f, opt_obj);
    } :
    function(arr, f, opt_obj) {
      var l = arr.length;  // must be fixed during loop... see docs
      var res = [];
      var resLength = 0;
      var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
      for (var i = 0; i < l; i++) {
        if (i in arr2) {
          var val = arr2[i];  // in case f mutates arr2
          if (f.call(/** @type {?} */ (opt_obj), val, i, arr)) {
            res[resLength++] = val;
          }
        }
      }
      return res;
    };


/**
 * Calls a function for each element in an array and inserts the result into a
 * new array.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-map}
 *
 * @param {IArrayLike<VALUE>|string} arr Array or array like object
 *     over which to iterate.
 * @param {function(this:THIS, VALUE, number, ?): RESULT} f The function to call
 *     for every element. This function takes 3 arguments (the element,
 *     the index and the array) and should return something. The result will be
 *     inserted into a new array.
 * @param {THIS=} opt_obj The object to be used as the value of 'this' within f.
 * @return {!Array<RESULT>} a new array with the results from f.
 * @template THIS, VALUE, RESULT
 */
goog.array.map = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.map) ?
    function(arr, f, opt_obj) {
      goog.asserts.assert(arr.length != null);

      return Array.prototype.map.call(arr, f, opt_obj);
    } :
    function(arr, f, opt_obj) {
      var l = arr.length;  // must be fixed during loop... see docs
      var res = new Array(l);
      var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
      for (var i = 0; i < l; i++) {
        if (i in arr2) {
          res[i] = f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr);
        }
      }
      return res;
    };


/**
 * Passes every element of an array into a function and accumulates the result.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-reduce}
 *
 * For example:
 * var a = [1, 2, 3, 4];
 * goog.array.reduce(a, function(r, v, i, arr) {return r + v;}, 0);
 * returns 10
 *
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {function(this:S, R, T, number, ?) : R} f The function to call for
 *     every element. This function
 *     takes 4 arguments (the function's previous result or the initial value,
 *     the value of the current array element, the current array index, and the
 *     array itself)
 *     function(previousValue, currentValue, index, array).
 * @param {?} val The initial value to pass into the function on the first call.
 * @param {S=} opt_obj  The object to be used as the value of 'this'
 *     within f.
 * @return {R} Result of evaluating f repeatedly across the values of the array.
 * @template T,S,R
 */
goog.array.reduce = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.reduce) ?
    function(arr, f, val, opt_obj) {
      goog.asserts.assert(arr.length != null);
      if (opt_obj) {
        f = goog.bind(f, opt_obj);
      }
      return Array.prototype.reduce.call(arr, f, val);
    } :
    function(arr, f, val, opt_obj) {
      var rval = val;
      goog.array.forEach(arr, function(val, index) {
        rval = f.call(/** @type {?} */ (opt_obj), rval, val, index, arr);
      });
      return rval;
    };


/**
 * Passes every element of an array into a function and accumulates the result,
 * starting from the last element and working towards the first.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-reduceright}
 *
 * For example:
 * var a = ['a', 'b', 'c'];
 * goog.array.reduceRight(a, function(r, v, i, arr) {return r + v;}, '');
 * returns 'cba'
 *
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, R, T, number, ?) : R} f The function to call for
 *     every element. This function
 *     takes 4 arguments (the function's previous result or the initial value,
 *     the value of the current array element, the current array index, and the
 *     array itself)
 *     function(previousValue, currentValue, index, array).
 * @param {?} val The initial value to pass into the function on the first call.
 * @param {S=} opt_obj The object to be used as the value of 'this'
 *     within f.
 * @return {R} Object returned as a result of evaluating f repeatedly across the
 *     values of the array.
 * @template T,S,R
 */
goog.array.reduceRight = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.reduceRight) ?
    function(arr, f, val, opt_obj) {
      goog.asserts.assert(arr.length != null);
      goog.asserts.assert(f != null);
      if (opt_obj) {
        f = goog.bind(f, opt_obj);
      }
      return Array.prototype.reduceRight.call(arr, f, val);
    } :
    function(arr, f, val, opt_obj) {
      var rval = val;
      goog.array.forEachRight(arr, function(val, index) {
        rval = f.call(/** @type {?} */ (opt_obj), rval, val, index, arr);
      });
      return rval;
    };


/**
 * Calls f for each element of an array. If any call returns true, some()
 * returns true (without checking the remaining elements). If all calls
 * return false, some() returns false.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-some}
 *
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call for
 *     for every element. This function takes 3 arguments (the element, the
 *     index and the array) and should return a boolean.
 * @param {S=} opt_obj  The object to be used as the value of 'this'
 *     within f.
 * @return {boolean} true if any element passes the test.
 * @template T,S
 */
goog.array.some = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.some) ?
    function(arr, f, opt_obj) {
      goog.asserts.assert(arr.length != null);

      return Array.prototype.some.call(arr, f, opt_obj);
    } :
    function(arr, f, opt_obj) {
      var l = arr.length;  // must be fixed during loop... see docs
      var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
      for (var i = 0; i < l; i++) {
        if (i in arr2 && f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr)) {
          return true;
        }
      }
      return false;
    };


/**
 * Call f for each element of an array. If all calls return true, every()
 * returns true. If any call returns false, every() returns false and
 * does not continue to check the remaining elements.
 *
 * See {@link http://tinyurl.com/developer-mozilla-org-array-every}
 *
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call for
 *     for every element. This function takes 3 arguments (the element, the
 *     index and the array) and should return a boolean.
 * @param {S=} opt_obj The object to be used as the value of 'this'
 *     within f.
 * @return {boolean} false if any element fails the test.
 * @template T,S
 */
goog.array.every = goog.NATIVE_ARRAY_PROTOTYPES &&
        (goog.array.ASSUME_NATIVE_FUNCTIONS || Array.prototype.every) ?
    function(arr, f, opt_obj) {
      goog.asserts.assert(arr.length != null);

      return Array.prototype.every.call(arr, f, opt_obj);
    } :
    function(arr, f, opt_obj) {
      var l = arr.length;  // must be fixed during loop... see docs
      var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
      for (var i = 0; i < l; i++) {
        if (i in arr2 && !f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr)) {
          return false;
        }
      }
      return true;
    };


/**
 * Counts the array elements that fulfill the predicate, i.e. for which the
 * callback function returns true. Skips holes in the array.
 *
 * @param {!IArrayLike<T>|string} arr Array or array like object
 *     over which to iterate.
 * @param {function(this: S, T, number, ?): boolean} f The function to call for
 *     every element. Takes 3 arguments (the element, the index and the array).
 * @param {S=} opt_obj The object to be used as the value of 'this' within f.
 * @return {number} The number of the matching elements.
 * @template T,S
 */
goog.array.count = function(arr, f, opt_obj) {
  var count = 0;
  goog.array.forEach(arr, function(element, index, arr) {
    if (f.call(/** @type {?} */ (opt_obj), element, index, arr)) {
      ++count;
    }
  }, opt_obj);
  return count;
};


/**
 * Search an array for the first element that satisfies a given condition and
 * return that element.
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call
 *     for every element. This function takes 3 arguments (the element, the
 *     index and the array) and should return a boolean.
 * @param {S=} opt_obj An optional "this" context for the function.
 * @return {T|null} The first array element that passes the test, or null if no
 *     element is found.
 * @template T,S
 */
goog.array.find = function(arr, f, opt_obj) {
  var i = goog.array.findIndex(arr, f, opt_obj);
  return i < 0 ? null : typeof arr === 'string' ? arr.charAt(i) : arr[i];
};


/**
 * Search an array for the first element that satisfies a given condition and
 * return its index.
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call for
 *     every element. This function
 *     takes 3 arguments (the element, the index and the array) and should
 *     return a boolean.
 * @param {S=} opt_obj An optional "this" context for the function.
 * @return {number} The index of the first array element that passes the test,
 *     or -1 if no element is found.
 * @template T,S
 */
goog.array.findIndex = function(arr, f, opt_obj) {
  var l = arr.length;  // must be fixed during loop... see docs
  var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
  for (var i = 0; i < l; i++) {
    if (i in arr2 && f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr)) {
      return i;
    }
  }
  return -1;
};


/**
 * Search an array (in reverse order) for the last element that satisfies a
 * given condition and return that element.
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call
 *     for every element. This function
 *     takes 3 arguments (the element, the index and the array) and should
 *     return a boolean.
 * @param {S=} opt_obj An optional "this" context for the function.
 * @return {T|null} The last array element that passes the test, or null if no
 *     element is found.
 * @template T,S
 */
goog.array.findRight = function(arr, f, opt_obj) {
  var i = goog.array.findIndexRight(arr, f, opt_obj);
  return i < 0 ? null : typeof arr === 'string' ? arr.charAt(i) : arr[i];
};


/**
 * Search an array (in reverse order) for the last element that satisfies a
 * given condition and return its index.
 * @param {IArrayLike<T>|string} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call
 *     for every element. This function
 *     takes 3 arguments (the element, the index and the array) and should
 *     return a boolean.
 * @param {S=} opt_obj An optional "this" context for the function.
 * @return {number} The index of the last array element that passes the test,
 *     or -1 if no element is found.
 * @template T,S
 */
goog.array.findIndexRight = function(arr, f, opt_obj) {
  var l = arr.length;  // must be fixed during loop... see docs
  var arr2 = (typeof arr === 'string') ? arr.split('') : arr;
  for (var i = l - 1; i >= 0; i--) {
    if (i in arr2 && f.call(/** @type {?} */ (opt_obj), arr2[i], i, arr)) {
      return i;
    }
  }
  return -1;
};


/**
 * Whether the array contains the given object.
 * @param {IArrayLike<?>|string} arr The array to test for the presence of the
 *     element.
 * @param {*} obj The object for which to test.
 * @return {boolean} true if obj is present.
 */
goog.array.contains = function(arr, obj) {
  return goog.array.indexOf(arr, obj) >= 0;
};


/**
 * Whether the array is empty.
 * @param {IArrayLike<?>|string} arr The array to test.
 * @return {boolean} true if empty.
 */
goog.array.isEmpty = function(arr) {
  return arr.length == 0;
};


/**
 * Clears the array.
 * @param {IArrayLike<?>} arr Array or array like object to clear.
 */
goog.array.clear = function(arr) {
  // For non real arrays we don't have the magic length so we delete the
  // indices.
  if (!Array.isArray(arr)) {
    for (var i = arr.length - 1; i >= 0; i--) {
      delete arr[i];
    }
  }
  arr.length = 0;
};


/**
 * Pushes an item into an array, if it's not already in the array.
 * @param {Array<T>} arr Array into which to insert the item.
 * @param {T} obj Value to add.
 * @template T
 */
goog.array.insert = function(arr, obj) {
  if (!goog.array.contains(arr, obj)) {
    arr.push(obj);
  }
};


/**
 * Inserts an object at the given index of the array.
 * @param {IArrayLike<?>} arr The array to modify.
 * @param {*} obj The object to insert.
 * @param {number=} opt_i The index at which to insert the object. If omitted,
 *      treated as 0. A negative index is counted from the end of the array.
 */
goog.array.insertAt = function(arr, obj, opt_i) {
  goog.array.splice(arr, opt_i, 0, obj);
};


/**
 * Inserts at the given index of the array, all elements of another array.
 * @param {IArrayLike<?>} arr The array to modify.
 * @param {IArrayLike<?>} elementsToAdd The array of elements to add.
 * @param {number=} opt_i The index at which to insert the object. If omitted,
 *      treated as 0. A negative index is counted from the end of the array.
 */
goog.array.insertArrayAt = function(arr, elementsToAdd, opt_i) {
  goog.partial(goog.array.splice, arr, opt_i, 0).apply(null, elementsToAdd);
};


/**
 * Inserts an object into an array before a specified object.
 * @param {Array<T>} arr The array to modify.
 * @param {T} obj The object to insert.
 * @param {T=} opt_obj2 The object before which obj should be inserted. If obj2
 *     is omitted or not found, obj is inserted at the end of the array.
 * @template T
 */
goog.array.insertBefore = function(arr, obj, opt_obj2) {
  var i;
  if (arguments.length == 2 || (i = goog.array.indexOf(arr, opt_obj2)) < 0) {
    arr.push(obj);
  } else {
    goog.array.insertAt(arr, obj, i);
  }
};


/**
 * Removes the first occurrence of a particular value from an array.
 * @param {IArrayLike<T>} arr Array from which to remove
 *     value.
 * @param {T} obj Object to remove.
 * @return {boolean} True if an element was removed.
 * @template T
 */
goog.array.remove = function(arr, obj) {
  var i = goog.array.indexOf(arr, obj);
  var rv;
  if ((rv = i >= 0)) {
    goog.array.removeAt(arr, i);
  }
  return rv;
};


/**
 * Removes the last occurrence of a particular value from an array.
 * @param {!IArrayLike<T>} arr Array from which to remove value.
 * @param {T} obj Object to remove.
 * @return {boolean} True if an element was removed.
 * @template T
 */
goog.array.removeLast = function(arr, obj) {
  var i = goog.array.lastIndexOf(arr, obj);
  if (i >= 0) {
    goog.array.removeAt(arr, i);
    return true;
  }
  return false;
};


/**
 * Removes from an array the element at index i
 * @param {IArrayLike<?>} arr Array or array like object from which to
 *     remove value.
 * @param {number} i The index to remove.
 * @return {boolean} True if an element was removed.
 */
goog.array.removeAt = function(arr, i) {
  goog.asserts.assert(arr.length != null);

  // use generic form of splice
  // splice returns the removed items and if successful the length of that
  // will be 1
  return Array.prototype.splice.call(arr, i, 1).length == 1;
};


/**
 * Removes the first value that satisfies the given condition.
 * @param {IArrayLike<T>} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call
 *     for every element. This function
 *     takes 3 arguments (the element, the index and the array) and should
 *     return a boolean.
 * @param {S=} opt_obj An optional "this" context for the function.
 * @return {boolean} True if an element was removed.
 * @template T,S
 */
goog.array.removeIf = function(arr, f, opt_obj) {
  var i = goog.array.findIndex(arr, f, opt_obj);
  if (i >= 0) {
    goog.array.removeAt(arr, i);
    return true;
  }
  return false;
};


/**
 * Removes all values that satisfy the given condition.
 * @param {IArrayLike<T>} arr Array or array
 *     like object over which to iterate.
 * @param {?function(this:S, T, number, ?) : boolean} f The function to call
 *     for every element. This function
 *     takes 3 arguments (the element, the index and the array) and should
 *     return a boolean.
 * @param {S=} opt_obj An optional "this" context for the function.
 * @return {number} The number of items removed
 * @template T,S
 */
goog.array.removeAllIf = function(arr, f, opt_obj) {
  var removedCount = 0;
  goog.array.forEachRight(arr, function(val, index) {
    if (f.call(/** @type {?} */ (opt_obj), val, index, arr)) {
      if (goog.array.removeAt(arr, index)) {
        removedCount++;
      }
    }
  });
  return removedCount;
};


/**
 * Returns a new array that is the result of joining the arguments.  If arrays
 * are passed then their items are added, however, if non-arrays are passed they
 * will be added to the return array as is.
 *
 * Note that ArrayLike objects will be added as is, rather than having their
 * items added.
 *
 * goog.array.concat([1, 2], [3, 4]) -> [1, 2, 3, 4]
 * goog.array.concat(0, [1, 2]) -> [0, 1, 2]
 * goog.array.concat([1, 2], null) -> [1, 2, null]
 *
 * There is bug in all current versions of IE (6, 7 and 8) where arrays created
 * in an iframe become corrupted soon (not immediately) after the iframe is
 * destroyed. This is common if loading data via goog.net.IframeIo, for example.
 * This corruption only affects the concat method which will start throwing
 * Catastrophic Errors (#-2147418113).
 *
 * See http://endoflow.com/scratch/corrupted-arrays.html for a test case.
 *
 * Internally goog.array should use this, so that all methods will continue to
 * work on these broken array objects.
 *
 * @param {...*} var_args Items to concatenate.  Arrays will have each item
 *     added, while primitives and objects will be added as is.
 * @return {!Array<?>} The new resultant array.
 */
goog.array.concat = function(var_args) {
  return Array.prototype.concat.apply([], arguments);
};


/**
 * Returns a new array that contains the contents of all the arrays passed.
 * @param {...!Array<T>} var_args
 * @return {!Array<T>}
 * @template T
 */
goog.array.join = function(var_args) {
  return Array.prototype.concat.apply([], arguments);
};


/**
 * Converts an object to an array.
 * @param {IArrayLike<T>|string} object  The object to convert to an
 *     array.
 * @return {!Array<T>} The object converted into an array. If object has a
 *     length property, every property indexed with a non-negative number
 *     less than length will be included in the result. If object does not
 *     have a length property, an empty array will be returned.
 * @template T
 */
goog.array.toArray = function(object) {
  var length = object.length;

  // If length is not a number the following is false. This case is kept for
  // backwards compatibility since there are callers that pass objects that are
  // not array like.
  if (length > 0) {
    var rv = new Array(length);
    for (var i = 0; i < length; i++) {
      rv[i] = object[i];
    }
    return rv;
  }
  return [];
};


/**
 * Does a shallow copy of an array.
 * @param {IArrayLike<T>|string} arr  Array or array-like object to
 *     clone.
 * @return {!Array<T>} Clone of the input array.
 * @template T
 */
goog.array.clone = goog.array.toArray;


/**
 * Extends an array with another array, element, or "array like" object.
 * This function operates 'in-place', it does not create a new Array.
 *
 * Example:
 * var a = [];
 * goog.array.extend(a, [0, 1]);
 * a; // [0, 1]
 * goog.array.extend(a, 2);
 * a; // [0, 1, 2]
 *
 * @param {Array<VALUE>} arr1  The array to modify.
 * @param {...(IArrayLike<VALUE>|VALUE)} var_args The elements or arrays of
 *     elements to add to arr1.
 * @template VALUE
 */
goog.array.extend = function(arr1, var_args) {
  for (var i = 1; i < arguments.length; i++) {
    var arr2 = arguments[i];
    if (goog.isArrayLike(arr2)) {
      var len1 = arr1.length || 0;
      var len2 = arr2.length || 0;
      arr1.length = len1 + len2;
      for (var j = 0; j < len2; j++) {
        arr1[len1 + j] = arr2[j];
      }
    } else {
      arr1.push(arr2);
    }
  }
};


/**
 * Adds or removes elements from an array. This is a generic version of Array
 * splice. This means that it might work on other objects similar to arrays,
 * such as the arguments object.
 *
 * @param {IArrayLike<T>} arr The array to modify.
 * @param {number|undefined} index The index at which to start changing the
 *     array. If not defined, treated as 0.
 * @param {number} howMany How many elements to remove (0 means no removal. A
 *     value below 0 is treated as zero and so is any other non number. Numbers
 *     are floored).
 * @param {...T} var_args Optional, additional elements to insert into the
 *     array.
 * @return {!Array<T>} the removed elements.
 * @template T
 */
goog.array.splice = function(arr, index, howMany, var_args) {
  goog.asserts.assert(arr.length != null);

  return Array.prototype.splice.apply(arr, goog.array.slice(arguments, 1));
};


/**
 * Returns a new array from a segment of an array. This is a generic version of
 * Array slice. This means that it might work on other objects similar to
 * arrays, such as the arguments object.
 *
 * @param {IArrayLike<T>|string} arr The array from
 * which to copy a segment.
 * @param {number} start The index of the first element to copy.
 * @param {number=} opt_end The index after the last element to copy.
 * @return {!Array<T>} A new array containing the specified segment of the
 *     original array.
 * @template T
 */
goog.array.slice = function(arr, start, opt_end) {
  goog.asserts.assert(arr.length != null);

  // passing 1 arg to slice is not the same as passing 2 where the second is
  // null or undefined (in that case the second argument is treated as 0).
  // we could use slice on the arguments object and then use apply instead of
  // testing the length
  if (arguments.length <= 2) {
    return Array.prototype.slice.call(arr, start);
  } else {
    return Array.prototype.slice.call(arr, start, opt_end);
  }
};


/**
 * Removes all duplicates from an array (retaining only the first
 * occurrence of each array element).  This function modifies the
 * array in place and doesn't change the order of the non-duplicate items.
 *
 * For objects, duplicates are identified as having the same unique ID as
 * defined by {@link goog.getUid}.
 *
 * Alternatively you can specify a custom hash function that returns a unique
 * value for each item in the array it should consider unique.
 *
 * Runtime: N,
 * Worstcase space: 2N (no dupes)
 *
 * @param {IArrayLike<T>} arr The array from which to remove
 *     duplicates.
 * @param {Array=} opt_rv An optional array in which to return the results,
 *     instead of performing the removal inplace.  If specified, the original
 *     array will remain unchanged.
 * @param {function(T):string=} opt_hashFn An optional function to use to
 *     apply to every item in the array. This function should return a unique
 *     value for each item in the array it should consider unique.
 * @template T
 */
goog.array.removeDuplicates = function(arr, opt_rv, opt_hashFn) {
  var returnArray = opt_rv || arr;
  var defaultHashFn = function(item) {
    // Prefix each type with a single character representing the type to
    // prevent conflicting keys (e.g. true and 'true').
    return goog.isObject(item) ? 'o' + goog.getUid(item) :
                                 (typeof item).charAt(0) + item;
  };
  var hashFn = opt_hashFn || defaultHashFn;

  var seen = {}, cursorInsert = 0, cursorRead = 0;
  while (cursorRead < arr.length) {
    var current = arr[cursorRead++];
    var key = hashFn(current);
    if (!Object.prototype.hasOwnProperty.call(seen, key)) {
      seen[key] = true;
      returnArray[cursorInsert++] = current;
    }
  }
  returnArray.length = cursorInsert;
};


/**
 * Searches the specified array for the specified target using the binary
 * search algorithm.  If no opt_compareFn is specified, elements are compared
 * using <code>goog.array.defaultCompare</code>, which compares the elements
 * using the built in < and > operators.  This will produce the expected
 * behavior for homogeneous arrays of String(s) and Number(s). The array
 * specified <b>must</b> be sorted in ascending order (as defined by the
 * comparison function).  If the array is not sorted, results are undefined.
 * If the array contains multiple instances of the specified target value, the
 * left-most instance will be found.
 *
 * Runtime: O(log n)
 *
 * @param {IArrayLike<VALUE>} arr The array to be searched.
 * @param {TARGET} target The sought value.
 * @param {function(TARGET, VALUE): number=} opt_compareFn Optional comparison
 *     function by which the array is ordered. Should take 2 arguments to
 *     compare, the target value and an element from your array, and return a
 *     negative number, zero, or a positive number depending on whether the
 *     first argument is less than, equal to, or greater than the second.
 * @return {number} Lowest index of the target value if found, otherwise
 *     (-(insertion point) - 1). The insertion point is where the value should
 *     be inserted into arr to preserve the sorted property.  Return value >= 0
 *     iff target is found.
 * @template TARGET, VALUE
 */
goog.array.binarySearch = function(arr, target, opt_compareFn) {
  return goog.array.binarySearch_(
      arr, opt_compareFn || goog.array.defaultCompare, false /* isEvaluator */,
      target);
};


/**
 * Selects an index in the specified array using the binary search algorithm.
 * The evaluator receives an element and determines whether the desired index
 * is before, at, or after it.  The evaluator must be consistent (formally,
 * goog.array.map(goog.array.map(arr, evaluator, opt_obj), goog.math.sign)
 * must be monotonically non-increasing).
 *
 * Runtime: O(log n)
 *
 * @param {IArrayLike<VALUE>} arr The array to be searched.
 * @param {function(this:THIS, VALUE, number, ?): number} evaluator
 *     Evaluator function that receives 3 arguments (the element, the index and
 *     the array). Should return a negative number, zero, or a positive number
 *     depending on whether the desired index is before, at, or after the
 *     element passed to it.
 * @param {THIS=} opt_obj The object to be used as the value of 'this'
 *     within evaluator.
 * @return {number} Index of the leftmost element matched by the evaluator, if
 *     such exists; otherwise (-(insertion point) - 1). The insertion point is
 *     the index of the first element for which the evaluator returns negative,
 *     or arr.length if no such element exists. The return value is non-negative
 *     iff a match is found.
 * @template THIS, VALUE
 */
goog.array.binarySelect = function(arr, evaluator, opt_obj) {
  return goog.array.binarySearch_(
      arr, evaluator, true /* isEvaluator */, undefined /* opt_target */,
      opt_obj);
};


/**
 * Implementation of a binary search algorithm which knows how to use both
 * comparison functions and evaluators. If an evaluator is provided, will call
 * the evaluator with the given optional data object, conforming to the
 * interface defined in binarySelect. Otherwise, if a comparison function is
 * provided, will call the comparison function against the given data object.
 *
 * This implementation purposefully does not use goog.bind or goog.partial for
 * performance reasons.
 *
 * Runtime: O(log n)
 *
 * @param {IArrayLike<?>} arr The array to be searched.
 * @param {function(?, ?, ?): number | function(?, ?): number} compareFn
 *     Either an evaluator or a comparison function, as defined by binarySearch
 *     and binarySelect above.
 * @param {boolean} isEvaluator Whether the function is an evaluator or a
 *     comparison function.
 * @param {?=} opt_target If the function is a comparison function, then
 *     this is the target to binary search for.
 * @param {Object=} opt_selfObj If the function is an evaluator, this is an
 *     optional this object for the evaluator.
 * @return {number} Lowest index of the target value if found, otherwise
 *     (-(insertion point) - 1). The insertion point is where the value should
 *     be inserted into arr to preserve the sorted property.  Return value >= 0
 *     iff target is found.
 * @private
 */
goog.array.binarySearch_ = function(
    arr, compareFn, isEvaluator, opt_target, opt_selfObj) {
  var left = 0;            // inclusive
  var right = arr.length;  // exclusive
  var found;
  while (left < right) {
    var middle = left + ((right - left) >>> 1);
    var compareResult;
    if (isEvaluator) {
      compareResult = compareFn.call(opt_selfObj, arr[middle], middle, arr);
    } else {
      // NOTE(dimvar): To avoid this cast, we'd have to use function overloading
      // for the type of binarySearch_, which the type system can't express yet.
      compareResult = /** @type {function(?, ?): number} */ (compareFn)(
          opt_target, arr[middle]);
    }
    if (compareResult > 0) {
      left = middle + 1;
    } else {
      right = middle;
      // We are looking for the lowest index so we can't return immediately.
      found = !compareResult;
    }
  }
  // left is the index if found, or the insertion point otherwise.
  // Avoiding bitwise not operator, as that causes a loss in precision for array
  // indexes outside the bounds of a 32-bit signed integer.  Array indexes have
  // a maximum value of 2^32-2 https://tc39.es/ecma262/#array-index
  return found ? left : -left - 1;
};


/**
 * Sorts the specified array into ascending order.  If no opt_compareFn is
 * specified, elements are compared using
 * <code>goog.array.defaultCompare</code>, which compares the elements using
 * the built in < and > operators.  This will produce the expected behavior
 * for homogeneous arrays of String(s) and Number(s), unlike the native sort,
 * but will give unpredictable results for heterogeneous lists of strings and
 * numbers with different numbers of digits.
 *
 * This sort is not guaranteed to be stable.
 *
 * Runtime: Same as <code>Array.prototype.sort</code>
 *
 * @param {Array<T>} arr The array to be sorted.
 * @param {?function(T,T):number=} opt_compareFn Optional comparison
 *     function by which the
 *     array is to be ordered. Should take 2 arguments to compare, and return a
 *     negative number, zero, or a positive number depending on whether the
 *     first argument is less than, equal to, or greater than the second.
 * @template T
 */
goog.array.sort = function(arr, opt_compareFn) {
  // TODO(arv): Update type annotation since null is not accepted.
  arr.sort(opt_compareFn || goog.array.defaultCompare);
};


/**
 * Sorts the specified array into ascending order in a stable way.  If no
 * opt_compareFn is specified, elements are compared using
 * <code>goog.array.defaultCompare</code>, which compares the elements using
 * the built in < and > operators.  This will produce the expected behavior
 * for homogeneous arrays of String(s) and Number(s).
 *
 * Runtime: Same as <code>Array.prototype.sort</code>, plus an additional
 * O(n) overhead of copying the array twice.
 *
 * @param {Array<T>} arr The array to be sorted.
 * @param {?function(T, T): number=} opt_compareFn Optional comparison function
 *     by which the array is to be ordered. Should take 2 arguments to compare,
 *     and return a negative number, zero, or a positive number depending on
 *     whether the first argument is less than, equal to, or greater than the
 *     second.
 * @template T
 */
goog.array.stableSort = function(arr, opt_compareFn) {
  var compArr = new Array(arr.length);
  for (var i = 0; i < arr.length; i++) {
    compArr[i] = {index: i, value: arr[i]};
  }
  var valueCompareFn = opt_compareFn || goog.array.defaultCompare;
  function stableCompareFn(obj1, obj2) {
    return valueCompareFn(obj1.value, obj2.value) || obj1.index - obj2.index;
  }
  goog.array.sort(compArr, stableCompareFn);
  for (var i = 0; i < arr.length; i++) {
    arr[i] = compArr[i].value;
  }
};


/**
 * Sort the specified array into ascending order based on item keys
 * returned by the specified key function.
 * If no opt_compareFn is specified, the keys are compared in ascending order
 * using <code>goog.array.defaultCompare</code>.
 *
 * Runtime: O(S(f(n)), where S is runtime of <code>goog.array.sort</code>
 * and f(n) is runtime of the key function.
 *
 * @param {Array<T>} arr The array to be sorted.
 * @param {function(T): K} keyFn Function taking array element and returning
 *     a key used for sorting this element.
 * @param {?function(K, K): number=} opt_compareFn Optional comparison function
 *     by which the keys are to be ordered. Should take 2 arguments to compare,
 *     and return a negative number, zero, or a positive number depending on
 *     whether the first argument is less than, equal to, or greater than the
 *     second.
 * @template T,K
 */
goog.array.sortByKey = function(arr, keyFn, opt_compareFn) {
  var keyCompareFn = opt_compareFn || goog.array.defaultCompare;
  goog.array.sort(
      arr, function(a, b) { return keyCompareFn(keyFn(a), keyFn(b)); });
};


/**
 * Sorts an array of objects by the specified object key and compare
 * function. If no compare function is provided, the key values are
 * compared in ascending order using <code>goog.array.defaultCompare</code>.
 * This won't work for keys that get renamed by the compiler. So use
 * {'foo': 1, 'bar': 2} rather than {foo: 1, bar: 2}.
 * @param {Array<Object>} arr An array of objects to sort.
 * @param {string} key The object key to sort by.
 * @param {Function=} opt_compareFn The function to use to compare key
 *     values.
 */
goog.array.sortObjectsByKey = function(arr, key, opt_compareFn) {
  goog.array.sortByKey(arr, function(obj) { return obj[key]; }, opt_compareFn);
};


/**
 * Tells if the array is sorted.
 * @param {!IArrayLike<T>} arr The array.
 * @param {?function(T,T):number=} opt_compareFn Function to compare the
 *     array elements.
 *     Should take 2 arguments to compare, and return a negative number, zero,
 *     or a positive number depending on whether the first argument is less
 *     than, equal to, or greater than the second.
 * @param {boolean=} opt_strict If true no equal elements are allowed.
 * @return {boolean} Whether the array is sorted.
 * @template T
 */
goog.array.isSorted = function(arr, opt_compareFn, opt_strict) {
  var compare = opt_compareFn || goog.array.defaultCompare;
  for (var i = 1; i < arr.length; i++) {
    var compareResult = compare(arr[i - 1], arr[i]);
    if (compareResult > 0 || compareResult == 0 && opt_strict) {
      return false;
    }
  }
  return true;
};


/**
 * Compares two arrays for equality. Two arrays are considered equal if they
 * have the same length and their corresponding elements are equal according to
 * the comparison function.
 *
 * @param {IArrayLike<?>} arr1 The first array to compare.
 * @param {IArrayLike<?>} arr2 The second array to compare.
 * @param {Function=} opt_equalsFn Optional comparison function.
 *     Should take 2 arguments to compare, and return true if the arguments
 *     are equal. Defaults to {@link goog.array.defaultCompareEquality} which
 *     compares the elements using the built-in '===' operator.
 * @return {boolean} Whether the two arrays are equal.
 */
goog.array.equals = function(arr1, arr2, opt_equalsFn) {
  if (!goog.isArrayLike(arr1) || !goog.isArrayLike(arr2) ||
      arr1.length != arr2.length) {
    return false;
  }
  var l = arr1.length;
  var equalsFn = opt_equalsFn || goog.array.defaultCompareEquality;
  for (var i = 0; i < l; i++) {
    if (!equalsFn(arr1[i], arr2[i])) {
      return false;
    }
  }
  return true;
};


/**
 * 3-way array compare function.
 * @param {!IArrayLike<VALUE>} arr1 The first array to
 *     compare.
 * @param {!IArrayLike<VALUE>} arr2 The second array to
 *     compare.
 * @param {function(VALUE, VALUE): number=} opt_compareFn Optional comparison
 *     function by which the array is to be ordered. Should take 2 arguments to
 *     compare, and return a negative number, zero, or a positive number
 *     depending on whether the first argument is less than, equal to, or
 *     greater than the second.
 * @return {number} Negative number, zero, or a positive number depending on
 *     whether the first argument is less than, equal to, or greater than the
 *     second.
 * @template VALUE
 */
goog.array.compare3 = function(arr1, arr2, opt_compareFn) {
  var compare = opt_compareFn || goog.array.defaultCompare;
  var l = Math.min(arr1.length, arr2.length);
  for (var i = 0; i < l; i++) {
    var result = compare(arr1[i], arr2[i]);
    if (result != 0) {
      return result;
    }
  }
  return goog.array.defaultCompare(arr1.length, arr2.length);
};


/**
 * Compares its two arguments for order, using the built in < and >
 * operators.
 * @param {VALUE} a The first object to be compared.
 * @param {VALUE} b The second object to be compared.
 * @return {number} A negative number, zero, or a positive number as the first
 *     argument is less than, equal to, or greater than the second,
 *     respectively.
 * @template VALUE
 */
goog.array.defaultCompare = function(a, b) {
  return a > b ? 1 : a < b ? -1 : 0;
};


/**
 * Compares its two arguments for inverse order, using the built in < and >
 * operators.
 * @param {VALUE} a The first object to be compared.
 * @param {VALUE} b The second object to be compared.
 * @return {number} A negative number, zero, or a positive number as the first
 *     argument is greater than, equal to, or less than the second,
 *     respectively.
 * @template VALUE
 */
goog.array.inverseDefaultCompare = function(a, b) {
  return -goog.array.defaultCompare(a, b);
};


/**
 * Compares its two arguments for equality, using the built in === operator.
 * @param {*} a The first object to compare.
 * @param {*} b The second object to compare.
 * @return {boolean} True if the two arguments are equal, false otherwise.
 */
goog.array.defaultCompareEquality = function(a, b) {
  return a === b;
};


/**
 * Inserts a value into a sorted array. The array is not modified if the
 * value is already present.
 * @param {IArrayLike<VALUE>} array The array to modify.
 * @param {VALUE} value The object to insert.
 * @param {function(VALUE, VALUE): number=} opt_compareFn Optional comparison
 *     function by which the array is ordered. Should take 2 arguments to
 *     compare, and return a negative number, zero, or a positive number
 *     depending on whether the first argument is less than, equal to, or
 *     greater than the second.
 * @return {boolean} True if an element was inserted.
 * @template VALUE
 */
goog.array.binaryInsert = function(array, value, opt_compareFn) {
  var index = goog.array.binarySearch(array, value, opt_compareFn);
  if (index < 0) {
    goog.array.insertAt(array, value, -(index + 1));
    return true;
  }
  return false;
};


/**
 * Removes a value from a sorted array.
 * @param {!IArrayLike<VALUE>} array The array to modify.
 * @param {VALUE} value The object to remove.
 * @param {function(VALUE, VALUE): number=} opt_compareFn Optional comparison
 *     function by which the array is ordered. Should take 2 arguments to
 *     compare, and return a negative number, zero, or a positive number
 *     depending on whether the first argument is less than, equal to, or
 *     greater than the second.
 * @return {boolean} True if an element was removed.
 * @template VALUE
 */
goog.array.binaryRemove = function(array, value, opt_compareFn) {
  var index = goog.array.binarySearch(array, value, opt_compareFn);
  return (index >= 0) ? goog.array.removeAt(array, index) : false;
};


/**
 * Splits an array into disjoint buckets according to a splitting function.
 * @param {IArrayLike<T>} array The array.
 * @param {function(this:S, T, number, !IArrayLike<T>):?} sorter Function to
 *     call for every element.  This takes 3 arguments (the element, the index
 *     and the array) and must return a valid object key (a string, number,
 *     etc), or undefined, if that object should not be placed in a bucket.
 * @param {S=} opt_obj The object to be used as the value of 'this' within
 *     sorter.
 * @return {!Object<!Array<T>>} An object, with keys being all of the unique
 *     return values of sorter, and values being arrays containing the items for
 *     which the splitter returned that key.
 * @template T,S
 */
goog.array.bucket = function(array, sorter, opt_obj) {
  var buckets = {};

  for (var i = 0; i < array.length; i++) {
    var value = array[i];
    var key = sorter.call(/** @type {?} */ (opt_obj), value, i, array);
    if (key !== undefined) {
      // Push the value to the right bucket, creating it if necessary.
      var bucket = buckets[key] || (buckets[key] = []);
      bucket.push(value);
    }
  }

  return buckets;
};


/**
 * Creates a new object built from the provided array and the key-generation
 * function.
 * @param {IArrayLike<T>} arr Array or array like object over
 *     which to iterate whose elements will be the values in the new object.
 * @param {?function(this:S, T, number, ?) : string} keyFunc The function to
 *     call for every element. This function takes 3 arguments (the element, the
 *     index and the array) and should return a string that will be used as the
 *     key for the element in the new object. If the function returns the same
 *     key for more than one element, the value for that key is
 *     implementation-defined.
 * @param {S=} opt_obj The object to be used as the value of 'this'
 *     within keyFunc.
 * @return {!Object<T>} The new object.
 * @template T,S
 */
goog.array.toObject = function(arr, keyFunc, opt_obj) {
  var ret = {};
  goog.array.forEach(arr, function(element, index) {
    ret[keyFunc.call(/** @type {?} */ (opt_obj), element, index, arr)] =
        element;
  });
  return ret;
};


/**
 * Creates a range of numbers in an arithmetic progression.
 *
 * Range takes 1, 2, or 3 arguments:
 * <pre>
 * range(5) is the same as range(0, 5, 1) and produces [0, 1, 2, 3, 4]
 * range(2, 5) is the same as range(2, 5, 1) and produces [2, 3, 4]
 * range(-2, -5, -1) produces [-2, -3, -4]
 * range(-2, -5, 1) produces [], since stepping by 1 wouldn't ever reach -5.
 * </pre>
 *
 * @param {number} startOrEnd The starting value of the range if an end argument
 *     is provided. Otherwise, the start value is 0, and this is the end value.
 * @param {number=} opt_end The optional end value of the range.
 * @param {number=} opt_step The step size between range values. Defaults to 1
 *     if opt_step is undefined or 0.
 * @return {!Array<number>} An array of numbers for the requested range. May be
 *     an empty array if adding the step would not converge toward the end
 *     value.
 */
goog.array.range = function(startOrEnd, opt_end, opt_step) {
  var array = [];
  var start = 0;
  var end = startOrEnd;
  var step = opt_step || 1;
  if (opt_end !== undefined) {
    start = startOrEnd;
    end = opt_end;
  }

  if (step * (end - start) < 0) {
    // Sign mismatch: start + step will never reach the end value.
    return [];
  }

  if (step > 0) {
    for (var i = start; i < end; i += step) {
      array.push(i);
    }
  } else {
    for (var i = start; i > end; i += step) {
      array.push(i);
    }
  }
  return array;
};


/**
 * Returns an array consisting of the given value repeated N times.
 *
 * @param {VALUE} value The value to repeat.
 * @param {number} n The repeat count.
 * @return {!Array<VALUE>} An array with the repeated value.
 * @template VALUE
 */
goog.array.repeat = function(value, n) {
  var array = [];
  for (var i = 0; i < n; i++) {
    array[i] = value;
  }
  return array;
};


/**
 * Returns an array consisting of every argument with all arrays
 * expanded in-place recursively.
 *
 * @param {...*} var_args The values to flatten.
 * @return {!Array<?>} An array containing the flattened values.
 */
goog.array.flatten = function(var_args) {
  var CHUNK_SIZE = 8192;

  var result = [];
  for (var i = 0; i < arguments.length; i++) {
    var element = arguments[i];
    if (Array.isArray(element)) {
      for (var c = 0; c < element.length; c += CHUNK_SIZE) {
        var chunk = goog.array.slice(element, c, c + CHUNK_SIZE);
        var recurseResult = goog.array.flatten.apply(null, chunk);
        for (var r = 0; r < recurseResult.length; r++) {
          result.push(recurseResult[r]);
        }
      }
    } else {
      result.push(element);
    }
  }
  return result;
};


/**
 * Rotates an array in-place. After calling this method, the element at
 * index i will be the element previously at index (i - n) %
 * array.length, for all values of i between 0 and array.length - 1,
 * inclusive.
 *
 * For example, suppose list comprises [t, a, n, k, s]. After invoking
 * rotate(array, 1) (or rotate(array, -4)), array will comprise [s, t, a, n, k].
 *
 * @param {!Array<T>} array The array to rotate.
 * @param {number} n The amount to rotate.
 * @return {!Array<T>} The array.
 * @template T
 */
goog.array.rotate = function(array, n) {
  goog.asserts.assert(array.length != null);

  if (array.length) {
    n %= array.length;
    if (n > 0) {
      Array.prototype.unshift.apply(array, array.splice(-n, n));
    } else if (n < 0) {
      Array.prototype.push.apply(array, array.splice(0, -n));
    }
  }
  return array;
};


/**
 * Moves one item of an array to a new position keeping the order of the rest
 * of the items. Example use case: keeping a list of JavaScript objects
 * synchronized with the corresponding list of DOM elements after one of the
 * elements has been dragged to a new position.
 * @param {!IArrayLike<?>} arr The array to modify.
 * @param {number} fromIndex Index of the item to move between 0 and
 *     {@code arr.length - 1}.
 * @param {number} toIndex Target index between 0 and {@code arr.length - 1}.
 */
goog.array.moveItem = function(arr, fromIndex, toIndex) {
  goog.asserts.assert(fromIndex >= 0 && fromIndex < arr.length);
  goog.asserts.assert(toIndex >= 0 && toIndex < arr.length);
  // Remove 1 item at fromIndex.
  var removedItems = Array.prototype.splice.call(arr, fromIndex, 1);
  // Insert the removed item at toIndex.
  Array.prototype.splice.call(arr, toIndex, 0, removedItems[0]);
  // We don't use goog.array.insertAt and goog.array.removeAt, because they're
  // significantly slower than splice.
};


/**
 * Creates a new array for which the element at position i is an array of the
 * ith element of the provided arrays.  The returned array will only be as long
 * as the shortest array provided; additional values are ignored.  For example,
 * the result of zipping [1, 2] and [3, 4, 5] is [[1,3], [2, 4]].
 *
 * This is similar to the zip() function in Python.  See {@link
 * http://docs.python.org/library/functions.html#zip}
 *
 * @param {...!IArrayLike<?>} var_args Arrays to be combined.
 * @return {!Array<!Array<?>>} A new array of arrays created from
 *     provided arrays.
 */
goog.array.zip = function(var_args) {
  if (!arguments.length) {
    return [];
  }
  var result = [];
  var minLen = arguments[0].length;
  for (var i = 1; i < arguments.length; i++) {
    if (arguments[i].length < minLen) {
      minLen = arguments[i].length;
    }
  }
  for (var i = 0; i < minLen; i++) {
    var value = [];
    for (var j = 0; j < arguments.length; j++) {
      value.push(arguments[j][i]);
    }
    result.push(value);
  }
  return result;
};


/**
 * Shuffles the values in the specified array using the Fisher-Yates in-place
 * shuffle (also known as the Knuth Shuffle). By default, calls Math.random()
 * and so resets the state of that random number generator. Similarly, may reset
 * the state of any other specified random number generator.
 *
 * Runtime: O(n)
 *
 * @param {!Array<?>} arr The array to be shuffled.
 * @param {function():number=} opt_randFn Optional random function to use for
 *     shuffling.
 *     Takes no arguments, and returns a random number on the interval [0, 1).
 *     Defaults to Math.random() using JavaScript's built-in Math library.
 */
goog.array.shuffle = function(arr, opt_randFn) {
  var randFn = opt_randFn || Math.random;

  for (var i = arr.length - 1; i > 0; i--) {
    // Choose a random array index in [0, i] (inclusive with i).
    var j = Math.floor(randFn() * (i + 1));

    var tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
  }
};


/**
 * Returns a new array of elements from arr, based on the indexes of elements
 * provided by index_arr. For example, the result of index copying
 * ['a', 'b', 'c'] with index_arr [1,0,0,2] is ['b', 'a', 'a', 'c'].
 *
 * @param {!IArrayLike<T>} arr The array to get a indexed copy from.
 * @param {!IArrayLike<number>} index_arr An array of indexes to get from arr.
 * @return {!Array<T>} A new array of elements from arr in index_arr order.
 * @template T
 */
goog.array.copyByIndex = function(arr, index_arr) {
  var result = [];
  goog.array.forEach(index_arr, function(index) { result.push(arr[index]); });
  return result;
};


/**
 * Maps each element of the input array into zero or more elements of the output
 * array.
 *
 * @param {!IArrayLike<VALUE>|string} arr Array or array like object
 *     over which to iterate.
 * @param {function(this:THIS, VALUE, number, ?): !Array<RESULT>} f The function
 *     to call for every element. This function takes 3 arguments (the element,
 *     the index and the array) and should return an array. The result will be
 *     used to extend a new array.
 * @param {THIS=} opt_obj The object to be used as the value of 'this' within f.
 * @return {!Array<RESULT>} a new array with the concatenation of all arrays
 *     returned from f.
 * @template THIS, VALUE, RESULT
 */
goog.array.concatMap = function(arr, f, opt_obj) {
  return goog.array.concat.apply([], goog.array.map(arr, f, opt_obj));
};

//third_party/javascript/closure/dom/htmlelement.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

goog.provide('goog.dom.HtmlElement');



/**
 * This subclass of HTMLElement is used when only a HTMLElement is possible and
 * not any of its subclasses. Normally, a type can refer to an instance of
 * itself or an instance of any subtype. More concretely, if HTMLElement is used
 * then the compiler must assume that it might still be e.g. HTMLScriptElement.
 * With this, the type check knows that it couldn't be any special element.
 *
 * @constructor
 * @extends {HTMLElement}
 */
goog.dom.HtmlElement = function() {};

//third_party/javascript/closure/dom/tagname.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Defines the goog.dom.TagName class. Its constants enumerate
 * all HTML tag names specified in either the W3C HTML 4.01 index of elements
 * or the HTML5.1 specification.
 *
 * References:
 * https://www.w3.org/TR/html401/index/elements.html
 * https://www.w3.org/TR/html51/dom.html#elements
 */
goog.provide('goog.dom.TagName');

goog.require('goog.dom.HtmlElement');


/**
 * A tag name with the type of the element stored in the generic.
 * @param {string} tagName
 * @constructor
 * @template T
 */
goog.dom.TagName = function(tagName) {
  /** @private {string} */
  this.tagName_ = tagName;
};


/**
 * Returns the tag name.
 * @return {string}
 * @override
 */
goog.dom.TagName.prototype.toString = function() {
  return this.tagName_;
};


// Closure Compiler unconditionally converts the following constants to their
// string value (goog.dom.TagName.A -> 'A'). These are the consequences:
// 1. Don't add any members or static members to goog.dom.TagName as they
//    couldn't be accessed after this optimization.
// 2. Keep the constant name and its string value the same:
//    goog.dom.TagName.X = new goog.dom.TagName('Y');
//    is converted to 'X', not 'Y'.


/** @type {!goog.dom.TagName<!HTMLAnchorElement>} */
goog.dom.TagName.A = new goog.dom.TagName('A');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.ABBR = new goog.dom.TagName('ABBR');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.ACRONYM = new goog.dom.TagName('ACRONYM');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.ADDRESS = new goog.dom.TagName('ADDRESS');


/** @type {!goog.dom.TagName<!HTMLAppletElement>} */
goog.dom.TagName.APPLET = new goog.dom.TagName('APPLET');


/** @type {!goog.dom.TagName<!HTMLAreaElement>} */
goog.dom.TagName.AREA = new goog.dom.TagName('AREA');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.ARTICLE = new goog.dom.TagName('ARTICLE');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.ASIDE = new goog.dom.TagName('ASIDE');


/** @type {!goog.dom.TagName<!HTMLAudioElement>} */
goog.dom.TagName.AUDIO = new goog.dom.TagName('AUDIO');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.B = new goog.dom.TagName('B');


/** @type {!goog.dom.TagName<!HTMLBaseElement>} */
goog.dom.TagName.BASE = new goog.dom.TagName('BASE');


/** @type {!goog.dom.TagName<!HTMLBaseFontElement>} */
goog.dom.TagName.BASEFONT = new goog.dom.TagName('BASEFONT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.BDI = new goog.dom.TagName('BDI');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.BDO = new goog.dom.TagName('BDO');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.BIG = new goog.dom.TagName('BIG');


/** @type {!goog.dom.TagName<!HTMLQuoteElement>} */
goog.dom.TagName.BLOCKQUOTE = new goog.dom.TagName('BLOCKQUOTE');


/** @type {!goog.dom.TagName<!HTMLBodyElement>} */
goog.dom.TagName.BODY = new goog.dom.TagName('BODY');


/** @type {!goog.dom.TagName<!HTMLBRElement>} */
goog.dom.TagName.BR = new goog.dom.TagName('BR');


/** @type {!goog.dom.TagName<!HTMLButtonElement>} */
goog.dom.TagName.BUTTON = new goog.dom.TagName('BUTTON');


/** @type {!goog.dom.TagName<!HTMLCanvasElement>} */
goog.dom.TagName.CANVAS = new goog.dom.TagName('CANVAS');


/** @type {!goog.dom.TagName<!HTMLTableCaptionElement>} */
goog.dom.TagName.CAPTION = new goog.dom.TagName('CAPTION');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.CENTER = new goog.dom.TagName('CENTER');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.CITE = new goog.dom.TagName('CITE');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.CODE = new goog.dom.TagName('CODE');


/** @type {!goog.dom.TagName<!HTMLTableColElement>} */
goog.dom.TagName.COL = new goog.dom.TagName('COL');


/** @type {!goog.dom.TagName<!HTMLTableColElement>} */
goog.dom.TagName.COLGROUP = new goog.dom.TagName('COLGROUP');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.COMMAND = new goog.dom.TagName('COMMAND');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.DATA = new goog.dom.TagName('DATA');


/** @type {!goog.dom.TagName<!HTMLDataListElement>} */
goog.dom.TagName.DATALIST = new goog.dom.TagName('DATALIST');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.DD = new goog.dom.TagName('DD');


/** @type {!goog.dom.TagName<!HTMLModElement>} */
goog.dom.TagName.DEL = new goog.dom.TagName('DEL');


/** @type {!goog.dom.TagName<!HTMLDetailsElement>} */
goog.dom.TagName.DETAILS = new goog.dom.TagName('DETAILS');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.DFN = new goog.dom.TagName('DFN');


/** @type {!goog.dom.TagName<!HTMLDialogElement>} */
goog.dom.TagName.DIALOG = new goog.dom.TagName('DIALOG');


/** @type {!goog.dom.TagName<!HTMLDirectoryElement>} */
goog.dom.TagName.DIR = new goog.dom.TagName('DIR');


/** @type {!goog.dom.TagName<!HTMLDivElement>} */
goog.dom.TagName.DIV = new goog.dom.TagName('DIV');


/** @type {!goog.dom.TagName<!HTMLDListElement>} */
goog.dom.TagName.DL = new goog.dom.TagName('DL');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.DT = new goog.dom.TagName('DT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.EM = new goog.dom.TagName('EM');


/** @type {!goog.dom.TagName<!HTMLEmbedElement>} */
goog.dom.TagName.EMBED = new goog.dom.TagName('EMBED');


/** @type {!goog.dom.TagName<!HTMLFieldSetElement>} */
goog.dom.TagName.FIELDSET = new goog.dom.TagName('FIELDSET');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.FIGCAPTION = new goog.dom.TagName('FIGCAPTION');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.FIGURE = new goog.dom.TagName('FIGURE');


/** @type {!goog.dom.TagName<!HTMLFontElement>} */
goog.dom.TagName.FONT = new goog.dom.TagName('FONT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.FOOTER = new goog.dom.TagName('FOOTER');


/** @type {!goog.dom.TagName<!HTMLFormElement>} */
goog.dom.TagName.FORM = new goog.dom.TagName('FORM');


/** @type {!goog.dom.TagName<!HTMLFrameElement>} */
goog.dom.TagName.FRAME = new goog.dom.TagName('FRAME');


/** @type {!goog.dom.TagName<!HTMLFrameSetElement>} */
goog.dom.TagName.FRAMESET = new goog.dom.TagName('FRAMESET');


/** @type {!goog.dom.TagName<!HTMLHeadingElement>} */
goog.dom.TagName.H1 = new goog.dom.TagName('H1');


/** @type {!goog.dom.TagName<!HTMLHeadingElement>} */
goog.dom.TagName.H2 = new goog.dom.TagName('H2');


/** @type {!goog.dom.TagName<!HTMLHeadingElement>} */
goog.dom.TagName.H3 = new goog.dom.TagName('H3');


/** @type {!goog.dom.TagName<!HTMLHeadingElement>} */
goog.dom.TagName.H4 = new goog.dom.TagName('H4');


/** @type {!goog.dom.TagName<!HTMLHeadingElement>} */
goog.dom.TagName.H5 = new goog.dom.TagName('H5');


/** @type {!goog.dom.TagName<!HTMLHeadingElement>} */
goog.dom.TagName.H6 = new goog.dom.TagName('H6');


/** @type {!goog.dom.TagName<!HTMLHeadElement>} */
goog.dom.TagName.HEAD = new goog.dom.TagName('HEAD');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.HEADER = new goog.dom.TagName('HEADER');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.HGROUP = new goog.dom.TagName('HGROUP');


/** @type {!goog.dom.TagName<!HTMLHRElement>} */
goog.dom.TagName.HR = new goog.dom.TagName('HR');


/** @type {!goog.dom.TagName<!HTMLHtmlElement>} */
goog.dom.TagName.HTML = new goog.dom.TagName('HTML');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.I = new goog.dom.TagName('I');


/** @type {!goog.dom.TagName<!HTMLIFrameElement>} */
goog.dom.TagName.IFRAME = new goog.dom.TagName('IFRAME');


/** @type {!goog.dom.TagName<!HTMLImageElement>} */
goog.dom.TagName.IMG = new goog.dom.TagName('IMG');


/** @type {!goog.dom.TagName<!HTMLInputElement>} */
goog.dom.TagName.INPUT = new goog.dom.TagName('INPUT');


/** @type {!goog.dom.TagName<!HTMLModElement>} */
goog.dom.TagName.INS = new goog.dom.TagName('INS');


/** @type {!goog.dom.TagName<!HTMLIsIndexElement>} */
goog.dom.TagName.ISINDEX = new goog.dom.TagName('ISINDEX');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.KBD = new goog.dom.TagName('KBD');


// HTMLKeygenElement is deprecated.
/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.KEYGEN = new goog.dom.TagName('KEYGEN');


/** @type {!goog.dom.TagName<!HTMLLabelElement>} */
goog.dom.TagName.LABEL = new goog.dom.TagName('LABEL');


/** @type {!goog.dom.TagName<!HTMLLegendElement>} */
goog.dom.TagName.LEGEND = new goog.dom.TagName('LEGEND');


/** @type {!goog.dom.TagName<!HTMLLIElement>} */
goog.dom.TagName.LI = new goog.dom.TagName('LI');


/** @type {!goog.dom.TagName<!HTMLLinkElement>} */
goog.dom.TagName.LINK = new goog.dom.TagName('LINK');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.MAIN = new goog.dom.TagName('MAIN');


/** @type {!goog.dom.TagName<!HTMLMapElement>} */
goog.dom.TagName.MAP = new goog.dom.TagName('MAP');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.MARK = new goog.dom.TagName('MARK');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.MATH = new goog.dom.TagName('MATH');


/** @type {!goog.dom.TagName<!HTMLMenuElement>} */
goog.dom.TagName.MENU = new goog.dom.TagName('MENU');


/** @type {!goog.dom.TagName<!HTMLMenuItemElement>} */
goog.dom.TagName.MENUITEM = new goog.dom.TagName('MENUITEM');


/** @type {!goog.dom.TagName<!HTMLMetaElement>} */
goog.dom.TagName.META = new goog.dom.TagName('META');


/** @type {!goog.dom.TagName<!HTMLMeterElement>} */
goog.dom.TagName.METER = new goog.dom.TagName('METER');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.NAV = new goog.dom.TagName('NAV');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.NOFRAMES = new goog.dom.TagName('NOFRAMES');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.NOSCRIPT = new goog.dom.TagName('NOSCRIPT');


/** @type {!goog.dom.TagName<!HTMLObjectElement>} */
goog.dom.TagName.OBJECT = new goog.dom.TagName('OBJECT');


/** @type {!goog.dom.TagName<!HTMLOListElement>} */
goog.dom.TagName.OL = new goog.dom.TagName('OL');


/** @type {!goog.dom.TagName<!HTMLOptGroupElement>} */
goog.dom.TagName.OPTGROUP = new goog.dom.TagName('OPTGROUP');


/** @type {!goog.dom.TagName<!HTMLOptionElement>} */
goog.dom.TagName.OPTION = new goog.dom.TagName('OPTION');


/** @type {!goog.dom.TagName<!HTMLOutputElement>} */
goog.dom.TagName.OUTPUT = new goog.dom.TagName('OUTPUT');


/** @type {!goog.dom.TagName<!HTMLParagraphElement>} */
goog.dom.TagName.P = new goog.dom.TagName('P');


/** @type {!goog.dom.TagName<!HTMLParamElement>} */
goog.dom.TagName.PARAM = new goog.dom.TagName('PARAM');


/** @type {!goog.dom.TagName<!HTMLPictureElement>} */
goog.dom.TagName.PICTURE = new goog.dom.TagName('PICTURE');


/** @type {!goog.dom.TagName<!HTMLPreElement>} */
goog.dom.TagName.PRE = new goog.dom.TagName('PRE');


/** @type {!goog.dom.TagName<!HTMLProgressElement>} */
goog.dom.TagName.PROGRESS = new goog.dom.TagName('PROGRESS');


/** @type {!goog.dom.TagName<!HTMLQuoteElement>} */
goog.dom.TagName.Q = new goog.dom.TagName('Q');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.RP = new goog.dom.TagName('RP');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.RT = new goog.dom.TagName('RT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.RTC = new goog.dom.TagName('RTC');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.RUBY = new goog.dom.TagName('RUBY');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.S = new goog.dom.TagName('S');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SAMP = new goog.dom.TagName('SAMP');


/** @type {!goog.dom.TagName<!HTMLScriptElement>} */
goog.dom.TagName.SCRIPT = new goog.dom.TagName('SCRIPT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SECTION = new goog.dom.TagName('SECTION');


/** @type {!goog.dom.TagName<!HTMLSelectElement>} */
goog.dom.TagName.SELECT = new goog.dom.TagName('SELECT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SMALL = new goog.dom.TagName('SMALL');


/** @type {!goog.dom.TagName<!HTMLSourceElement>} */
goog.dom.TagName.SOURCE = new goog.dom.TagName('SOURCE');


/** @type {!goog.dom.TagName<!HTMLSpanElement>} */
goog.dom.TagName.SPAN = new goog.dom.TagName('SPAN');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.STRIKE = new goog.dom.TagName('STRIKE');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.STRONG = new goog.dom.TagName('STRONG');


/** @type {!goog.dom.TagName<!HTMLStyleElement>} */
goog.dom.TagName.STYLE = new goog.dom.TagName('STYLE');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SUB = new goog.dom.TagName('SUB');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SUMMARY = new goog.dom.TagName('SUMMARY');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SUP = new goog.dom.TagName('SUP');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.SVG = new goog.dom.TagName('SVG');


/** @type {!goog.dom.TagName<!HTMLTableElement>} */
goog.dom.TagName.TABLE = new goog.dom.TagName('TABLE');


/** @type {!goog.dom.TagName<!HTMLTableSectionElement>} */
goog.dom.TagName.TBODY = new goog.dom.TagName('TBODY');


/** @type {!goog.dom.TagName<!HTMLTableCellElement>} */
goog.dom.TagName.TD = new goog.dom.TagName('TD');


/** @type {!goog.dom.TagName<!HTMLTemplateElement>} */
goog.dom.TagName.TEMPLATE = new goog.dom.TagName('TEMPLATE');


/** @type {!goog.dom.TagName<!HTMLTextAreaElement>} */
goog.dom.TagName.TEXTAREA = new goog.dom.TagName('TEXTAREA');


/** @type {!goog.dom.TagName<!HTMLTableSectionElement>} */
goog.dom.TagName.TFOOT = new goog.dom.TagName('TFOOT');


/** @type {!goog.dom.TagName<!HTMLTableCellElement>} */
goog.dom.TagName.TH = new goog.dom.TagName('TH');


/** @type {!goog.dom.TagName<!HTMLTableSectionElement>} */
goog.dom.TagName.THEAD = new goog.dom.TagName('THEAD');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.TIME = new goog.dom.TagName('TIME');


/** @type {!goog.dom.TagName<!HTMLTitleElement>} */
goog.dom.TagName.TITLE = new goog.dom.TagName('TITLE');


/** @type {!goog.dom.TagName<!HTMLTableRowElement>} */
goog.dom.TagName.TR = new goog.dom.TagName('TR');


/** @type {!goog.dom.TagName<!HTMLTrackElement>} */
goog.dom.TagName.TRACK = new goog.dom.TagName('TRACK');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.TT = new goog.dom.TagName('TT');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.U = new goog.dom.TagName('U');


/** @type {!goog.dom.TagName<!HTMLUListElement>} */
goog.dom.TagName.UL = new goog.dom.TagName('UL');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.VAR = new goog.dom.TagName('VAR');


/** @type {!goog.dom.TagName<!HTMLVideoElement>} */
goog.dom.TagName.VIDEO = new goog.dom.TagName('VIDEO');


/** @type {!goog.dom.TagName<!goog.dom.HtmlElement>} */
goog.dom.TagName.WBR = new goog.dom.TagName('WBR');

//third_party/javascript/closure/object/object.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Utilities for manipulating objects/maps/hashes.
 */

goog.provide('goog.object');


/**
 * Whether two values are not observably distinguishable. This
 * correctly detects that 0 is not the same as -0 and two NaNs are
 * practically equivalent.
 *
 * The implementation is as suggested by harmony:egal proposal.
 *
 * @param {*} v The first value to compare.
 * @param {*} v2 The second value to compare.
 * @return {boolean} Whether two values are not observably distinguishable.
 * @see http://wiki.ecmascript.org/doku.php?id=harmony:egal
 */
goog.object.is = function(v, v2) {
  if (v === v2) {
    // 0 === -0, but they are not identical.
    // We need the cast because the compiler requires that v2 is a
    // number (although 1/v2 works with non-number). We cast to ? to
    // stop the compiler from type-checking this statement.
    return v !== 0 || 1 / v === 1 / /** @type {?} */ (v2);
  }

  // NaN is non-reflexive: NaN !== NaN, although they are identical.
  return v !== v && v2 !== v2;
};


/**
 * Calls a function for each element in an object/map/hash.
 *
 * @param {Object<K,V>} obj The object over which to iterate.
 * @param {function(this:T,V,?,Object<K,V>):?} f The function to call
 *     for every element. This function takes 3 arguments (the value, the
 *     key and the object) and the return value is ignored.
 * @param {T=} opt_obj This is used as the 'this' object within f.
 * @template T,K,V
 */
goog.object.forEach = function(obj, f, opt_obj) {
  for (const key in obj) {
    f.call(/** @type {?} */ (opt_obj), obj[key], key, obj);
  }
};


/**
 * Calls a function for each element in an object/map/hash. If that call returns
 * true, adds the element to a new object.
 *
 * @param {Object<K,V>} obj The object over which to iterate.
 * @param {function(this:T,V,?,Object<K,V>):boolean} f The function to call
 *     for every element. This
 *     function takes 3 arguments (the value, the key and the object)
 *     and should return a boolean. If the return value is true the
 *     element is added to the result object. If it is false the
 *     element is not included.
 * @param {T=} opt_obj This is used as the 'this' object within f.
 * @return {!Object<K,V>} a new object in which only elements that passed the
 *     test are present.
 * @template T,K,V
 */
goog.object.filter = function(obj, f, opt_obj) {
  const res = {};
  for (const key in obj) {
    if (f.call(/** @type {?} */ (opt_obj), obj[key], key, obj)) {
      res[key] = obj[key];
    }
  }
  return res;
};


/**
 * For every element in an object/map/hash calls a function and inserts the
 * result into a new object.
 *
 * @param {Object<K,V>} obj The object over which to iterate.
 * @param {function(this:T,V,?,Object<K,V>):R} f The function to call
 *     for every element. This function
 *     takes 3 arguments (the value, the key and the object)
 *     and should return something. The result will be inserted
 *     into a new object.
 * @param {T=} opt_obj This is used as the 'this' object within f.
 * @return {!Object<K,R>} a new object with the results from f.
 * @template T,K,V,R
 */
goog.object.map = function(obj, f, opt_obj) {
  const res = {};
  for (const key in obj) {
    res[key] = f.call(/** @type {?} */ (opt_obj), obj[key], key, obj);
  }
  return res;
};


/**
 * Calls a function for each element in an object/map/hash. If any
 * call returns true, returns true (without checking the rest). If
 * all calls return false, returns false.
 *
 * @param {Object<K,V>} obj The object to check.
 * @param {function(this:T,V,?,Object<K,V>):boolean} f The function to
 *     call for every element. This function
 *     takes 3 arguments (the value, the key and the object) and should
 *     return a boolean.
 * @param {T=} opt_obj This is used as the 'this' object within f.
 * @return {boolean} true if any element passes the test.
 * @template T,K,V
 */
goog.object.some = function(obj, f, opt_obj) {
  for (const key in obj) {
    if (f.call(/** @type {?} */ (opt_obj), obj[key], key, obj)) {
      return true;
    }
  }
  return false;
};


/**
 * Calls a function for each element in an object/map/hash. If
 * all calls return true, returns true. If any call returns false, returns
 * false at this point and does not continue to check the remaining elements.
 *
 * @param {Object<K,V>} obj The object to check.
 * @param {?function(this:T,V,?,Object<K,V>):boolean} f The function to
 *     call for every element. This function
 *     takes 3 arguments (the value, the key and the object) and should
 *     return a boolean.
 * @param {T=} opt_obj This is used as the 'this' object within f.
 * @return {boolean} false if any element fails the test.
 * @template T,K,V
 */
goog.object.every = function(obj, f, opt_obj) {
  for (const key in obj) {
    if (!f.call(/** @type {?} */ (opt_obj), obj[key], key, obj)) {
      return false;
    }
  }
  return true;
};


/**
 * Returns the number of key-value pairs in the object map.
 *
 * @param {Object} obj The object for which to get the number of key-value
 *     pairs.
 * @return {number} The number of key-value pairs in the object map.
 */
goog.object.getCount = function(obj) {
  let rv = 0;
  for (const key in obj) {
    rv++;
  }
  return rv;
};


/**
 * Returns one key from the object map, if any exists.
 * For map literals the returned key will be the first one in most of the
 * browsers (a know exception is Konqueror).
 *
 * @param {Object} obj The object to pick a key from.
 * @return {string|undefined} The key or undefined if the object is empty.
 */
goog.object.getAnyKey = function(obj) {
  for (const key in obj) {
    return key;
  }
};


/**
 * Returns one value from the object map, if any exists.
 * For map literals the returned value will be the first one in most of the
 * browsers (a know exception is Konqueror).
 *
 * @param {Object<K,V>} obj The object to pick a value from.
 * @return {V|undefined} The value or undefined if the object is empty.
 * @template K,V
 */
goog.object.getAnyValue = function(obj) {
  for (const key in obj) {
    return obj[key];
  }
};


/**
 * Whether the object/hash/map contains the given object as a value.
 * An alias for goog.object.containsValue(obj, val).
 *
 * @param {Object<K,V>} obj The object in which to look for val.
 * @param {V} val The object for which to check.
 * @return {boolean} true if val is present.
 * @template K,V
 */
goog.object.contains = function(obj, val) {
  return goog.object.containsValue(obj, val);
};


/**
 * Returns the values of the object/map/hash.
 *
 * @param {Object<K,V>} obj The object from which to get the values.
 * @return {!Array<V>} The values in the object/map/hash.
 * @template K,V
 */
goog.object.getValues = function(obj) {
  const res = [];
  let i = 0;
  for (const key in obj) {
    res[i++] = obj[key];
  }
  return res;
};


/**
 * Returns the keys of the object/map/hash.
 *
 * @param {Object} obj The object from which to get the keys.
 * @return {!Array<string>} Array of property keys.
 */
goog.object.getKeys = function(obj) {
  const res = [];
  let i = 0;
  for (const key in obj) {
    res[i++] = key;
  }
  return res;
};


/**
 * Get a value from an object multiple levels deep.  This is useful for
 * pulling values from deeply nested objects, such as JSON responses.
 * Example usage: getValueByKeys(jsonObj, 'foo', 'entries', 3)
 *
 * @param {!Object} obj An object to get the value from.  Can be array-like.
 * @param {...(string|number|!IArrayLike<number|string>)}
 *     var_args A number of keys
 *     (as strings, or numbers, for array-like objects).  Can also be
 *     specified as a single array of keys.
 * @return {*} The resulting value.  If, at any point, the value for a key
 *     in the current object is null or undefined, returns undefined.
 */
goog.object.getValueByKeys = function(obj, var_args) {
  const isArrayLike = goog.isArrayLike(var_args);
  const keys = isArrayLike ?
      /** @type {!IArrayLike<number|string>} */ (var_args) :
      arguments;

  // Start with the 2nd parameter for the variable parameters syntax.
  for (let i = isArrayLike ? 0 : 1; i < keys.length; i++) {
    if (obj == null) return undefined;
    obj = obj[keys[i]];
  }

  return obj;
};


/**
 * Whether the object/map/hash contains the given key.
 *
 * @param {Object} obj The object in which to look for key.
 * @param {?} key The key for which to check.
 * @return {boolean} true If the map contains the key.
 */
goog.object.containsKey = function(obj, key) {
  return obj !== null && key in obj;
};


/**
 * Whether the object/map/hash contains the given value. This is O(n).
 *
 * @param {Object<K,V>} obj The object in which to look for val.
 * @param {V} val The value for which to check.
 * @return {boolean} true If the map contains the value.
 * @template K,V
 */
goog.object.containsValue = function(obj, val) {
  for (const key in obj) {
    if (obj[key] == val) {
      return true;
    }
  }
  return false;
};


/**
 * Searches an object for an element that satisfies the given condition and
 * returns its key.
 * @param {Object<K,V>} obj The object to search in.
 * @param {function(this:T,V,string,Object<K,V>):boolean} f The
 *      function to call for every element. Takes 3 arguments (the value,
 *     the key and the object) and should return a boolean.
 * @param {T=} opt_this An optional "this" context for the function.
 * @return {string|undefined} The key of an element for which the function
 *     returns true or undefined if no such element is found.
 * @template T,K,V
 */
goog.object.findKey = function(obj, f, opt_this) {
  for (const key in obj) {
    if (f.call(/** @type {?} */ (opt_this), obj[key], key, obj)) {
      return key;
    }
  }
  return undefined;
};


/**
 * Searches an object for an element that satisfies the given condition and
 * returns its value.
 * @param {Object<K,V>} obj The object to search in.
 * @param {function(this:T,V,string,Object<K,V>):boolean} f The function
 *     to call for every element. Takes 3 arguments (the value, the key
 *     and the object) and should return a boolean.
 * @param {T=} opt_this An optional "this" context for the function.
 * @return {V} The value of an element for which the function returns true or
 *     undefined if no such element is found.
 * @template T,K,V
 */
goog.object.findValue = function(obj, f, opt_this) {
  const key = goog.object.findKey(obj, f, opt_this);
  return key && obj[key];
};


/**
 * Whether the object/map/hash is empty.
 *
 * @param {Object} obj The object to test.
 * @return {boolean} true if obj is empty.
 */
goog.object.isEmpty = function(obj) {
  for (const key in obj) {
    return false;
  }
  return true;
};


/**
 * Removes all key value pairs from the object/map/hash.
 *
 * @param {Object} obj The object to clear.
 */
goog.object.clear = function(obj) {
  for (const i in obj) {
    delete obj[i];
  }
};


/**
 * Removes a key-value pair based on the key.
 *
 * @param {Object} obj The object from which to remove the key.
 * @param {?} key The key to remove.
 * @return {boolean} Whether an element was removed.
 */
goog.object.remove = function(obj, key) {
  let rv;
  if (rv = key in /** @type {!Object} */ (obj)) {
    delete obj[key];
  }
  return rv;
};


/**
 * Adds a key-value pair to the object. Throws an exception if the key is
 * already in use. Use set if you want to change an existing pair.
 *
 * @param {Object<K,V>} obj The object to which to add the key-value pair.
 * @param {string} key The key to add.
 * @param {V} val The value to add.
 * @template K,V
 */
goog.object.add = function(obj, key, val) {
  if (obj !== null && key in obj) {
    throw new Error('The object already contains the key "' + key + '"');
  }
  goog.object.set(obj, key, val);
};


/**
 * Returns the value for the given key.
 *
 * @param {Object<K,V>} obj The object from which to get the value.
 * @param {string} key The key for which to get the value.
 * @param {R=} opt_val The value to return if no item is found for the given
 *     key (default is undefined).
 * @return {V|R|undefined} The value for the given key.
 * @template K,V,R
 */
goog.object.get = function(obj, key, opt_val) {
  if (obj !== null && key in obj) {
    return obj[key];
  }
  return opt_val;
};


/**
 * Adds a key-value pair to the object/map/hash.
 *
 * @param {Object<K,V>} obj The object to which to add the key-value pair.
 * @param {string} key The key to add.
 * @param {V} value The value to add.
 * @template K,V
 */
goog.object.set = function(obj, key, value) {
  obj[key] = value;
};


/**
 * Adds a key-value pair to the object/map/hash if it doesn't exist yet.
 *
 * @param {Object<K,V>} obj The object to which to add the key-value pair.
 * @param {string} key The key to add.
 * @param {V} value The value to add if the key wasn't present.
 * @return {V} The value of the entry at the end of the function.
 * @template K,V
 */
goog.object.setIfUndefined = function(obj, key, value) {
  return key in /** @type {!Object} */ (obj) ? obj[key] : (obj[key] = value);
};


/**
 * Sets a key and value to an object if the key is not set. The value will be
 * the return value of the given function. If the key already exists, the
 * object will not be changed and the function will not be called (the function
 * will be lazily evaluated -- only called if necessary).
 *
 * This function is particularly useful when used with an `Object` which is
 * acting as a cache.
 *
 * @param {!Object<K,V>} obj The object to which to add the key-value pair.
 * @param {string} key The key to add.
 * @param {function():V} f The value to add if the key wasn't present.
 * @return {V} The value of the entry at the end of the function.
 * @template K,V
 */
goog.object.setWithReturnValueIfNotSet = function(obj, key, f) {
  if (key in obj) {
    return obj[key];
  }

  const val = f();
  obj[key] = val;
  return val;
};


/**
 * Compares two objects for equality using === on the values.
 *
 * @param {!Object<K,V>} a
 * @param {!Object<K,V>} b
 * @return {boolean}
 * @template K,V
 */
goog.object.equals = function(a, b) {
  for (const k in a) {
    if (!(k in b) || a[k] !== b[k]) {
      return false;
    }
  }
  for (const k in b) {
    if (!(k in a)) {
      return false;
    }
  }
  return true;
};


/**
 * Returns a shallow clone of the object.
 *
 * @param {Object<K,V>} obj Object to clone.
 * @return {!Object<K,V>} Clone of the input object.
 * @template K,V
 */
goog.object.clone = function(obj) {
  // We cannot use the prototype trick because a lot of methods depend on where
  // the actual key is set.

  const res = {};
  for (const key in obj) {
    res[key] = obj[key];
  }
  return res;
  // We could also use goog.mixin but I wanted this to be independent from that.
};


/**
 * Clones a value. The input may be an Object, Array, or basic type. Objects and
 * arrays will be cloned recursively.
 *
 * WARNINGS:
 * <code>goog.object.unsafeClone</code> does not detect reference loops. Objects
 * that refer to themselves will cause infinite recursion.
 *
 * <code>goog.object.unsafeClone</code> is unaware of unique identifiers, and
 * copies UIDs created by <code>getUid</code> into cloned results.
 *
 * @param {T} obj The value to clone.
 * @return {T} A clone of the input value.
 * @template T
 */
goog.object.unsafeClone = function(obj) {
  const type = goog.typeOf(obj);
  if (type == 'object' || type == 'array') {
    if (goog.isFunction(obj.clone)) {
      return obj.clone();
    }
    const clone = type == 'array' ? [] : {};
    for (const key in obj) {
      clone[key] = goog.object.unsafeClone(obj[key]);
    }
    return clone;
  }

  return obj;
};


/**
 * Returns a new object in which all the keys and values are interchanged
 * (keys become values and values become keys). If multiple keys map to the
 * same value, the chosen transposed value is implementation-dependent.
 *
 * @param {Object} obj The object to transpose.
 * @return {!Object} The transposed object.
 */
goog.object.transpose = function(obj) {
  const transposed = {};
  for (const key in obj) {
    transposed[obj[key]] = key;
  }
  return transposed;
};


/**
 * The names of the fields that are defined on Object.prototype.
 * @type {Array<string>}
 * @private
 */
goog.object.PROTOTYPE_FIELDS_ = [
  'constructor', 'hasOwnProperty', 'isPrototypeOf', 'propertyIsEnumerable',
  'toLocaleString', 'toString', 'valueOf'
];


/**
 * Extends an object with another object.
 * This operates 'in-place'; it does not create a new Object.
 *
 * Example:
 * var o = {};
 * goog.object.extend(o, {a: 0, b: 1});
 * o; // {a: 0, b: 1}
 * goog.object.extend(o, {b: 2, c: 3});
 * o; // {a: 0, b: 2, c: 3}
 *
 * @param {Object} target The object to modify. Existing properties will be
 *     overwritten if they are also present in one of the objects in
 *     `var_args`.
 * @param {...(Object|null|undefined)} var_args The objects from which values
 *     will be copied.
 * @deprecated Prefer Object.assign
 */
goog.object.extend = function(target, var_args) {
  let key;
  let source;
  for (let i = 1; i < arguments.length; i++) {
    source = arguments[i];
    for (key in source) {
      target[key] = source[key];
    }

    // For IE the for-in-loop does not contain any properties that are not
    // enumerable on the prototype object (for example isPrototypeOf from
    // Object.prototype) and it will also not include 'replace' on objects that
    // extend String and change 'replace' (not that it is common for anyone to
    // extend anything except Object).

    for (let j = 0; j < goog.object.PROTOTYPE_FIELDS_.length; j++) {
      key = goog.object.PROTOTYPE_FIELDS_[j];
      if (Object.prototype.hasOwnProperty.call(source, key)) {
        target[key] = source[key];
      }
    }
  }
};


/**
 * Creates a new object built from the key-value pairs provided as arguments.
 * @param {...*} var_args If only one argument is provided and it is an array
 *     then this is used as the arguments, otherwise even arguments are used as
 *     the property names and odd arguments are used as the property values.
 * @return {!Object} The new object.
 * @throws {Error} If there are uneven number of arguments or there is only one
 *     non array argument.
 */
goog.object.create = function(var_args) {
  const argLength = arguments.length;
  if (argLength == 1 && Array.isArray(arguments[0])) {
    return goog.object.create.apply(null, arguments[0]);
  }

  if (argLength % 2) {
    throw new Error('Uneven number of arguments');
  }

  const rv = {};
  for (let i = 0; i < argLength; i += 2) {
    rv[arguments[i]] = arguments[i + 1];
  }
  return rv;
};


/**
 * Creates a new object where the property names come from the arguments but
 * the value is always set to true
 * @param {...*} var_args If only one argument is provided and it is an array
 *     then this is used as the arguments, otherwise the arguments are used
 *     as the property names.
 * @return {!Object} The new object.
 */
goog.object.createSet = function(var_args) {
  const argLength = arguments.length;
  if (argLength == 1 && Array.isArray(arguments[0])) {
    return goog.object.createSet.apply(null, arguments[0]);
  }

  const rv = {};
  for (let i = 0; i < argLength; i++) {
    rv[arguments[i]] = true;
  }
  return rv;
};


/**
 * Creates an immutable view of the underlying object, if the browser
 * supports immutable objects.
 *
 * In default mode, writes to this view will fail silently. In strict mode,
 * they will throw an error.
 *
 * @param {!Object<K,V>} obj An object.
 * @return {!Object<K,V>} An immutable view of that object, or the
 *     original object if this browser does not support immutables.
 * @template K,V
 */
goog.object.createImmutableView = function(obj) {
  let result = obj;
  if (Object.isFrozen && !Object.isFrozen(obj)) {
    result = Object.create(obj);
    Object.freeze(result);
  }
  return result;
};


/**
 * @param {!Object} obj An object.
 * @return {boolean} Whether this is an immutable view of the object.
 */
goog.object.isImmutableView = function(obj) {
  return !!Object.isFrozen && Object.isFrozen(obj);
};


/**
 * Get all properties names on a given Object regardless of enumerability.
 *
 * <p> If the browser does not support `Object.getOwnPropertyNames` nor
 * `Object.getPrototypeOf` then this is equivalent to using
 * `goog.object.getKeys`
 *
 * @param {?Object} obj The object to get the properties of.
 * @param {boolean=} opt_includeObjectPrototype Whether properties defined on
 *     `Object.prototype` should be included in the result.
 * @param {boolean=} opt_includeFunctionPrototype Whether properties defined on
 *     `Function.prototype` should be included in the result.
 * @return {!Array<string>}
 * @public
 */
goog.object.getAllPropertyNames = function(
    obj, opt_includeObjectPrototype, opt_includeFunctionPrototype) {
  if (!obj) {
    return [];
  }

  // Naively use a for..in loop to get the property names if the browser doesn't
  // support any other APIs for getting it.
  if (!Object.getOwnPropertyNames || !Object.getPrototypeOf) {
    return goog.object.getKeys(obj);
  }

  const visitedSet = {};

  // Traverse the prototype chain and add all properties to the visited set.
  let proto = obj;
  while (proto &&
         (proto !== Object.prototype || !!opt_includeObjectPrototype) &&
         (proto !== Function.prototype || !!opt_includeFunctionPrototype)) {
    const names = Object.getOwnPropertyNames(proto);
    for (let i = 0; i < names.length; i++) {
      visitedSet[names[i]] = true;
    }
    proto = Object.getPrototypeOf(proto);
  }

  return goog.object.getKeys(visitedSet);
};


/**
 * Given a ES5 or ES6 class reference, return its super class / super
 * constructor.
 *
 * This should be used in rare cases where you need to walk up the inheritance
 * tree (this is generally a bad idea). But this work with ES5 and ES6 classes,
 * unlike relying on the superClass_ property.
 *
 * Note: To start walking up the hierarchy from an instance call this with its
 * `constructor` property; e.g. `getSuperClass(instance.constructor)`.
 *
 * @param {function(new: ?)} constructor
 * @return {?Object}
 */
goog.object.getSuperClass = function(constructor) {
  var proto = Object.getPrototypeOf(constructor.prototype);
  return proto && proto.constructor;
};

//third_party/javascript/closure/dom/tags.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Utilities for HTML element tag names.
 */
goog.provide('goog.dom.tags');

goog.require('goog.object');


/**
 * The void elements specified by
 * http://www.w3.org/TR/html-markup/syntax.html#void-elements.
 * @const @private {!Object<string, boolean>}
 */
goog.dom.tags.VOID_TAGS_ = goog.object.createSet(
    'area', 'base', 'br', 'col', 'command', 'embed', 'hr', 'img', 'input',
    'keygen', 'link', 'meta', 'param', 'source', 'track', 'wbr');


/**
 * Checks whether the tag is void (with no contents allowed and no legal end
 * tag), for example 'br'.
 * @param {string} tagName The tag name in lower case.
 * @return {boolean}
 */
goog.dom.tags.isVoidTag = function(tagName) {
  return goog.dom.tags.VOID_TAGS_[tagName] === true;
};

//third_party/javascript/closure/html/trustedtypes.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Policy to convert strings to Trusted Types. See
 * https://github.com/WICG/trusted-types for details.
 *
 * @visibility {//javascript/closure:__pkg__}
 * @visibility {//javascript/closure/bin/sizetests:__pkg__}
 * @visibility {//javascript/closure/dom:__pkg__}
 */

goog.provide('goog.html.trustedtypes');

/** @package @const {?TrustedTypePolicy} */
goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY =
    goog.TRUSTED_TYPES_POLICY_NAME ?
    goog.createTrustedTypesPolicy(goog.TRUSTED_TYPES_POLICY_NAME + '#html') :
    null;

//third_party/javascript/closure/string/typedstring.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

goog.provide('goog.string.TypedString');



/**
 * Wrapper for strings that conform to a data type or language.
 *
 * Implementations of this interface are wrappers for strings, and typically
 * associate a type contract with the wrapped string.  Concrete implementations
 * of this interface may choose to implement additional run-time type checking,
 * see for example `goog.html.SafeHtml`. If available, client code that
 * needs to ensure type membership of an object should use the type's function
 * to assert type membership, such as `goog.html.SafeHtml.unwrap`.
 * @interface
 */
goog.string.TypedString = function() {};


/**
 * Interface marker of the TypedString interface.
 *
 * This property can be used to determine at runtime whether or not an object
 * implements this interface.  All implementations of this interface set this
 * property to `true`.
 * @type {boolean}
 */
goog.string.TypedString.prototype.implementsGoogStringTypedString;


/**
 * Retrieves this wrapped string's value.
 * @return {string} The wrapped string's value.
 */
goog.string.TypedString.prototype.getTypedStringValue;

//third_party/javascript/closure/string/const.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

goog.provide('goog.string.Const');

goog.require('goog.asserts');
goog.require('goog.string.TypedString');



/**
 * Wrapper for compile-time-constant strings.
 *
 * Const is a wrapper for strings that can only be created from program
 * constants (i.e., string literals).  This property relies on a custom Closure
 * compiler check that `goog.string.Const.from` is only invoked on
 * compile-time-constant expressions.
 *
 * Const is useful in APIs whose correct and secure use requires that certain
 * arguments are not attacker controlled: Compile-time constants are inherently
 * under the control of the application and not under control of external
 * attackers, and hence are safe to use in such contexts.
 *
 * Instances of this type must be created via its factory method
 * `goog.string.Const.from` and not by invoking its constructor.  The
 * constructor intentionally takes no parameters and the type is immutable;
 * hence only a default instance corresponding to the empty string can be
 * obtained via constructor invocation.  Use goog.string.Const.EMPTY
 * instead of using this constructor to get an empty Const string.
 *
 * @see goog.string.Const#from
 * @constructor
 * @final
 * @struct
 * @implements {goog.string.TypedString}
 * @param {Object=} opt_token package-internal implementation detail.
 * @param {string=} opt_content package-internal implementation detail.
 */
goog.string.Const = function(opt_token, opt_content) {
  /**
   * The wrapped value of this Const object.  The field has a purposely ugly
   * name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @private {string}
   */
  this.stringConstValueWithSecurityContract__googStringSecurityPrivate_ =
      ((opt_token ===
        goog.string.Const.GOOG_STRING_CONSTRUCTOR_TOKEN_PRIVATE_) &&
       opt_content) ||
      '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.string.Const#unwrap
   * @const {!Object}
   * @private
   */
  this.STRING_CONST_TYPE_MARKER__GOOG_STRING_SECURITY_PRIVATE_ =
      goog.string.Const.TYPE_MARKER_;
};


/**
 * @override
 * @const
 */
goog.string.Const.prototype.implementsGoogStringTypedString = true;


/**
 * Returns this Const's value as a string.
 *
 * IMPORTANT: In code where it is security-relevant that an object's type is
 * indeed `goog.string.Const`, use `goog.string.Const.unwrap`
 * instead of this method.
 *
 * @see goog.string.Const#unwrap
 * @override
 */
goog.string.Const.prototype.getTypedStringValue = function() {
  return this.stringConstValueWithSecurityContract__googStringSecurityPrivate_;
};


if (goog.DEBUG) {
  /**
   * Returns a debug-string representation of this value.
   *
   * To obtain the actual string value wrapped inside an object of this type,
   * use `goog.string.Const.unwrap`.
   *
   * @see goog.string.Const#unwrap
   * @override
   */
  goog.string.Const.prototype.toString = function() {
    return 'Const{' +
        this.stringConstValueWithSecurityContract__googStringSecurityPrivate_ +
        '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed an instance
 * of `goog.string.Const`, and returns its value.
 * @param {!goog.string.Const} stringConst The object to extract from.
 * @return {string} The Const object's contained string, unless the run-time
 *     type check fails. In that case, `unwrap` returns an innocuous
 *     string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.string.Const.unwrap = function(stringConst) {
  // Perform additional run-time type-checking to ensure that stringConst is
  // indeed an instance of the expected type.  This provides some additional
  // protection against security bugs due to application code that disables type
  // checks.
  if (stringConst instanceof goog.string.Const &&
      stringConst.constructor === goog.string.Const &&
      stringConst.STRING_CONST_TYPE_MARKER__GOOG_STRING_SECURITY_PRIVATE_ ===
          goog.string.Const.TYPE_MARKER_) {
    return stringConst
        .stringConstValueWithSecurityContract__googStringSecurityPrivate_;
  } else {
    goog.asserts.fail(
        'expected object of type Const, got \'' + stringConst + '\'');
    return 'type_error:Const';
  }
};


/**
 * Creates a Const object from a compile-time constant string.
 *
 * It is illegal to invoke this function on an expression whose
 * compile-time-constant value cannot be determined by the Closure compiler.
 *
 * Correct invocations include,
 * <pre>
 *   var s = goog.string.Const.from('hello');
 *   var t = goog.string.Const.from('hello' + 'world');
 * </pre>
 *
 * In contrast, the following are illegal:
 * <pre>
 *   var s = goog.string.Const.from(getHello());
 *   var t = goog.string.Const.from('hello' + world);
 * </pre>
 *
 * @param {string} s A constant string from which to create a Const.
 * @return {!goog.string.Const} A Const object initialized to stringConst.
 */
goog.string.Const.from = function(s) {
  return new goog.string.Const(
      goog.string.Const.GOOG_STRING_CONSTRUCTOR_TOKEN_PRIVATE_, s);
};

/**
 * Type marker for the Const type, used to implement additional run-time
 * type checking.
 * @const {!Object}
 * @private
 */
goog.string.Const.TYPE_MARKER_ = {};

/**
 * @type {!Object}
 * @private
 * @const
 */
goog.string.Const.GOOG_STRING_CONSTRUCTOR_TOKEN_PRIVATE_ = {};

/**
 * A Const instance wrapping the empty string.
 * @const {!goog.string.Const}
 */
goog.string.Const.EMPTY = goog.string.Const.from('');

//third_party/javascript/closure/html/safescript.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview The SafeScript type and its builders.
 *
 * TODO(xtof): Link to document stating type contract.
 */

goog.provide('goog.html.SafeScript');

goog.require('goog.asserts');
goog.require('goog.html.trustedtypes');
goog.require('goog.string.Const');
goog.require('goog.string.TypedString');



/**
 * A string-like object which represents JavaScript code and that carries the
 * security type contract that its value, as a string, will not cause execution
 * of unconstrained attacker controlled code (XSS) when evaluated as JavaScript
 * in a browser.
 *
 * Instances of this type must be created via the factory method
 * `goog.html.SafeScript.fromConstant` and not by invoking its
 * constructor. The constructor intentionally takes no parameters and the type
 * is immutable; hence only a default instance corresponding to the empty string
 * can be obtained via constructor invocation.
 *
 * A SafeScript's string representation can safely be interpolated as the
 * content of a script element within HTML. The SafeScript string should not be
 * escaped before interpolation.
 *
 * Note that the SafeScript might contain text that is attacker-controlled but
 * that text should have been interpolated with appropriate escaping,
 * sanitization and/or validation into the right location in the script, such
 * that it is highly constrained in its effect (for example, it had to match a
 * set of whitelisted words).
 *
 * A SafeScript can be constructed via security-reviewed unchecked
 * conversions. In this case producers of SafeScript must ensure themselves that
 * the SafeScript does not contain unsafe script. Note in particular that
 * {@code &lt;} is dangerous, even when inside JavaScript strings, and so should
 * always be forbidden or JavaScript escaped in user controlled input. For
 * example, if {@code &lt;/script&gt;&lt;script&gt;evil&lt;/script&gt;"} were
 * interpolated inside a JavaScript string, it would break out of the context
 * of the original script element and `evil` would execute. Also note
 * that within an HTML script (raw text) element, HTML character references,
 * such as "&lt;" are not allowed. See
 * http://www.w3.org/TR/html5/scripting-1.html#restrictions-for-contents-of-script-elements.
 *
 * @see goog.html.SafeScript#fromConstant
 * @constructor
 * @final
 * @struct
 * @implements {goog.string.TypedString}
 */
goog.html.SafeScript = function() {
  /**
   * The contained value of this SafeScript.  The field has a purposely
   * ugly name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @private {!TrustedScript|string}
   */
  this.privateDoNotAccessOrElseSafeScriptWrappedValue_ = '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.html.SafeScript#unwrap
   * @const {!Object}
   * @private
   */
  this.SAFE_SCRIPT_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ =
      goog.html.SafeScript.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_;
};


/**
 * @override
 * @const
 */
goog.html.SafeScript.prototype.implementsGoogStringTypedString = true;


/**
 * Type marker for the SafeScript type, used to implement additional
 * run-time type checking.
 * @const {!Object}
 * @private
 */
goog.html.SafeScript.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ = {};


/**
 * Creates a SafeScript object from a compile-time constant string.
 *
 * @param {!goog.string.Const} script A compile-time-constant string from which
 *     to create a SafeScript.
 * @return {!goog.html.SafeScript} A SafeScript object initialized to
 *     `script`.
 */
goog.html.SafeScript.fromConstant = function(script) {
  var scriptString = goog.string.Const.unwrap(script);
  if (scriptString.length === 0) {
    return goog.html.SafeScript.EMPTY;
  }
  return goog.html.SafeScript.createSafeScriptSecurityPrivateDoNotAccessOrElse(
      scriptString);
};


/**
 * Creates a SafeScript from a compile-time constant string but with arguments
 * that can vary at run-time. The code argument should be formatted as an
 * inline function (see example below). The arguments will be JSON-encoded and
 * provided as input to the function specified in code.
 *
 * Example Usage:
 *
 *     let safeScript = SafeScript.fromConstantAndArgs(
 *         Const.from('function(arg1, arg2) { doSomething(arg1, arg2); }'),
 *         arg1,
 *         arg2);
 *
 * This produces a SafeScript equivalent to the following:
 *
 *     (function(arg1, arg2) { doSomething(arg1, arg2); })("value1", "value2");
 *
 * @param {!goog.string.Const} code
 * @param {...*} var_args
 * @return {!goog.html.SafeScript}
 */
goog.html.SafeScript.fromConstantAndArgs = function(code, var_args) {
  var args = [];
  for (var i = 1; i < arguments.length; i++) {
    args.push(goog.html.SafeScript.stringify_(arguments[i]));
  }
  return goog.html.SafeScript.createSafeScriptSecurityPrivateDoNotAccessOrElse(
      '(' + goog.string.Const.unwrap(code) + ')(' + args.join(', ') + ');');
};


/**
 * Creates a SafeScript JSON representation from anything that could be passed
 * to JSON.stringify.
 * @param {*} val
 * @return {!goog.html.SafeScript}
 */
goog.html.SafeScript.fromJson = function(val) {
  return goog.html.SafeScript.createSafeScriptSecurityPrivateDoNotAccessOrElse(
      goog.html.SafeScript.stringify_(val));
};


/**
 * Returns this SafeScript's value as a string.
 *
 * IMPORTANT: In code where it is security relevant that an object's type is
 * indeed `SafeScript`, use `goog.html.SafeScript.unwrap` instead of
 * this method. If in doubt, assume that it's security relevant. In particular,
 * note that goog.html functions which return a goog.html type do not guarantee
 * the returned instance is of the right type. For example:
 *
 * <pre>
 * var fakeSafeHtml = new String('fake');
 * fakeSafeHtml.__proto__ = goog.html.SafeHtml.prototype;
 * var newSafeHtml = goog.html.SafeHtml.htmlEscape(fakeSafeHtml);
 * // newSafeHtml is just an alias for fakeSafeHtml, it's passed through by
 * // goog.html.SafeHtml.htmlEscape() as fakeSafeHtml
 * // instanceof goog.html.SafeHtml.
 * </pre>
 *
 * @see goog.html.SafeScript#unwrap
 * @override
 */
goog.html.SafeScript.prototype.getTypedStringValue = function() {
  return this.privateDoNotAccessOrElseSafeScriptWrappedValue_.toString();
};


if (goog.DEBUG) {
  /**
   * Returns a debug string-representation of this value.
   *
   * To obtain the actual string value wrapped in a SafeScript, use
   * `goog.html.SafeScript.unwrap`.
   *
   * @see goog.html.SafeScript#unwrap
   * @override
   */
  goog.html.SafeScript.prototype.toString = function() {
    return 'SafeScript{' +
        this.privateDoNotAccessOrElseSafeScriptWrappedValue_ + '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed a
 * SafeScript object, and returns its value.
 *
 * @param {!goog.html.SafeScript} safeScript The object to extract from.
 * @return {string} The safeScript object's contained string, unless
 *     the run-time type check fails. In that case, `unwrap` returns an
 *     innocuous string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.html.SafeScript.unwrap = function(safeScript) {
  return goog.html.SafeScript.unwrapTrustedScript(safeScript).toString();
};


/**
 * Unwraps value as TrustedScript if supported or as a string if not.
 * @param {!goog.html.SafeScript} safeScript
 * @return {!TrustedScript|string}
 * @see goog.html.SafeScript.unwrap
 */
goog.html.SafeScript.unwrapTrustedScript = function(safeScript) {
  // Perform additional Run-time type-checking to ensure that
  // safeScript is indeed an instance of the expected type.  This
  // provides some additional protection against security bugs due to
  // application code that disables type checks.
  // Specifically, the following checks are performed:
  // 1. The object is an instance of the expected type.
  // 2. The object is not an instance of a subclass.
  // 3. The object carries a type marker for the expected type. "Faking" an
  // object requires a reference to the type marker, which has names intended
  // to stand out in code reviews.
  if (safeScript instanceof goog.html.SafeScript &&
      safeScript.constructor === goog.html.SafeScript &&
      safeScript.SAFE_SCRIPT_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ ===
          goog.html.SafeScript.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_) {
    return safeScript.privateDoNotAccessOrElseSafeScriptWrappedValue_;
  } else {
    goog.asserts.fail('expected object of type SafeScript, got \'' +
        safeScript + '\' of type ' + goog.typeOf(safeScript));
    return 'type_error:SafeScript';
  }
};


/**
 * Converts the given value to a embeddabel JSON string and returns it. The
 * resulting string can be embedded in HTML because the '<' character is
 * encoded.
 *
 * @param {*} val
 * @return {string}
 * @private
 */
goog.html.SafeScript.stringify_ = function(val) {
  var json = JSON.stringify(val);
  return json.replace(/</g, '\\x3c');
};

/**
 * Package-internal utility method to create SafeScript instances.
 *
 * @param {string} script The string to initialize the SafeScript object with.
 * @return {!goog.html.SafeScript} The initialized SafeScript object.
 * @package
 */
goog.html.SafeScript.createSafeScriptSecurityPrivateDoNotAccessOrElse =
    function(script) {
  return new goog.html.SafeScript().initSecurityPrivateDoNotAccessOrElse_(
      script);
};


/**
 * Called from createSafeScriptSecurityPrivateDoNotAccessOrElse(). This
 * method exists only so that the compiler can dead code eliminate static
 * fields (like EMPTY) when they're not accessed.
 * @param {string} script
 * @return {!goog.html.SafeScript}
 * @private
 */
goog.html.SafeScript.prototype.initSecurityPrivateDoNotAccessOrElse_ = function(
    script) {
  this.privateDoNotAccessOrElseSafeScriptWrappedValue_ =
      goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY ?
      goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY.createScript(
          script) :
      script;
  return this;
};


/**
 * A SafeScript instance corresponding to the empty string.
 * @const {!goog.html.SafeScript}
 */
goog.html.SafeScript.EMPTY =
    goog.html.SafeScript.createSafeScriptSecurityPrivateDoNotAccessOrElse('');

//third_party/javascript/closure/fs/url.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Wrapper for URL and its createObjectUrl and revokeObjectUrl
 * methods that are part of the HTML5 File API.
 */

goog.provide('goog.fs.url');


/**
 * Creates a blob URL for a blob object.
 * Throws an error if the browser does not support Object Urls.
 *
 * @param {!File|!Blob|!MediaSource|!MediaStream} obj The object for which
 *   to create the URL.
 * @return {string} The URL for the object.
 */
goog.fs.url.createObjectUrl = function(obj) {
  return goog.fs.url.getUrlObject_().createObjectURL(obj);
};


/**
 * Revokes a URL created by {@link goog.fs.url.createObjectUrl}.
 * Throws an error if the browser does not support Object Urls.
 *
 * @param {string} url The URL to revoke.
 */
goog.fs.url.revokeObjectUrl = function(url) {
  goog.fs.url.getUrlObject_().revokeObjectURL(url);
};


/**
 * @record
 * @private
 */
goog.fs.url.UrlObject_ = function() {};

/**
 * @param {!File|!Blob|!MediaSource|!MediaStream} arg
 * @return {string}
 */
goog.fs.url.UrlObject_.prototype.createObjectURL = function(arg) {};

/**
 * @param {string} s
 */
goog.fs.url.UrlObject_.prototype.revokeObjectURL = function(s) {};


/**
 * Get the object that has the createObjectURL and revokeObjectURL functions for
 * this browser.
 *
 * @return {!goog.fs.url.UrlObject_} The object for this browser.
 * @private
 */
goog.fs.url.getUrlObject_ = function() {
  const urlObject = goog.fs.url.findUrlObject_();
  if (urlObject != null) {
    return urlObject;
  } else {
    throw new Error('This browser doesn\'t seem to support blob URLs');
  }
};


/**
 * Finds the object that has the createObjectURL and revokeObjectURL functions
 * for this browser.
 *
 * @return {?goog.fs.url.UrlObject_} The object for this browser or null if the
 *     browser does not support Object Urls.
 * @private
 */
goog.fs.url.findUrlObject_ = function() {
  // This is what the spec says to do
  // http://dev.w3.org/2006/webapi/FileAPI/#dfn-createObjectURL
  if (goog.global.URL !== undefined &&
      goog.global.URL.createObjectURL !== undefined) {
    return /** @type {!goog.fs.url.UrlObject_} */ (goog.global.URL);
    // This is what Chrome does (as of 10.0.648.6 dev)
  } else if (
      goog.global.webkitURL !== undefined &&
      goog.global.webkitURL.createObjectURL !== undefined) {
    return /** @type {!goog.fs.url.UrlObject_} */ (goog.global.webkitURL);
    // This is what the spec used to say to do
  } else if (goog.global.createObjectURL !== undefined) {
    return /** @type {!goog.fs.url.UrlObject_} */ (goog.global);
  } else {
    return null;
  }
};


/**
 * Checks whether this browser supports Object Urls. If not, calls to
 * createObjectUrl and revokeObjectUrl will result in an error.
 *
 * @return {boolean} True if this browser supports Object Urls.
 */
goog.fs.url.browserSupportsObjectUrls = function() {
  return goog.fs.url.findUrlObject_() != null;
};

//third_party/javascript/closure/fs/blob.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Wrappers for the HTML5 File API. These wrappers closely mirror
 * the underlying APIs, but use Closure-style events and Deferred return values.
 * Their existence also makes it possible to mock the FileSystem API for testing
 * in browsers that don't support it natively.
 *
 * When adding public functions to anything under this namespace, be sure to add
 * its mock counterpart to goog.testing.fs.
 */

goog.provide('goog.fs.blob');

goog.require('goog.array');

/**
 * Concatenates one or more values together and converts them to a Blob.
 *
 * @param {...(string|!Blob|!ArrayBuffer)} var_args The values that will make up
 *     the resulting blob.
 * @return {!Blob} The blob.
 */
goog.fs.blob.getBlob = function(var_args) {
  var BlobBuilder = goog.global.BlobBuilder || goog.global.WebKitBlobBuilder;

  if (BlobBuilder !== undefined) {
    var bb = new BlobBuilder();
    for (var i = 0; i < arguments.length; i++) {
      bb.append(arguments[i]);
    }
    return bb.getBlob();
  } else {
    return goog.fs.blob.getBlobWithProperties(goog.array.toArray(arguments));
  }
};


/**
 * Creates a blob with the given properties.
 * See https://developer.mozilla.org/en-US/docs/Web/API/Blob for more details.
 *
 * @param {!Array<string|!Blob|!ArrayBuffer>} parts The values that will make up
 *     the resulting blob (subset supported by both BlobBuilder.append() and
 *     Blob constructor).
 * @param {string=} opt_type The MIME type of the Blob.
 * @param {string=} opt_endings Specifies how strings containing newlines are to
 *     be written out.
 * @return {!Blob} The blob.
 */
goog.fs.blob.getBlobWithProperties = function(parts, opt_type, opt_endings) {
  var BlobBuilder = goog.global.BlobBuilder || goog.global.WebKitBlobBuilder;

  if (BlobBuilder !== undefined) {
    var bb = new BlobBuilder();
    for (var i = 0; i < parts.length; i++) {
      bb.append(parts[i], opt_endings);
    }
    return bb.getBlob(opt_type);
  } else if (goog.global.Blob !== undefined) {
    var properties = {};
    if (opt_type) {
      properties['type'] = opt_type;
    }
    if (opt_endings) {
      properties['endings'] = opt_endings;
    }
    return new Blob(parts, properties);
  } else {
    throw new Error('This browser doesn\'t seem to support creating Blobs');
  }
};

//third_party/javascript/closure/html/trustedresourceurl.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview The TrustedResourceUrl type and its builders.
 *
 * TODO(xtof): Link to document stating type contract.
 */

goog.provide('goog.html.TrustedResourceUrl');

goog.require('goog.asserts');
goog.require('goog.fs.blob');
goog.require('goog.fs.url');
goog.require('goog.html.SafeScript');
goog.require('goog.html.trustedtypes');
goog.require('goog.i18n.bidi.Dir');
goog.require('goog.i18n.bidi.DirectionalString');
goog.require('goog.string.Const');
goog.require('goog.string.TypedString');



/**
 * A URL which is under application control and from which script, CSS, and
 * other resources that represent executable code, can be fetched.
 *
 * Given that the URL can only be constructed from strings under application
 * control and is used to load resources, bugs resulting in a malformed URL
 * should not have a security impact and are likely to be easily detectable
 * during testing. Given the wide number of non-RFC compliant URLs in use,
 * stricter validation could prevent some applications from being able to use
 * this type.
 *
 * Instances of this type must be created via the factory method,
 * (`fromConstant`, `fromConstants`, `format` or
 * `formatWithParams`), and not by invoking its constructor. The constructor
 * is organized in a way that only methods from that file can call it and
 * initialize with non-empty values. Anyone else calling constructor will
 * get default instance with empty value.
 *
 * @see goog.html.TrustedResourceUrl#fromConstant
 * @constructor
 * @final
 * @struct
 * @implements {goog.i18n.bidi.DirectionalString}
 * @implements {goog.string.TypedString}
 * @param {!Object=} opt_token package-internal implementation detail.
 * @param {!TrustedScriptURL|string=} opt_content package-internal
 *     implementation detail.
 */
goog.html.TrustedResourceUrl = function(opt_token, opt_content) {
  /**
   * The contained value of this TrustedResourceUrl.  The field has a purposely
   * ugly name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @const
   * @private {!TrustedScriptURL|string}
   */
  this.privateDoNotAccessOrElseTrustedResourceUrlWrappedValue_ =
      ((opt_token ===
        goog.html.TrustedResourceUrl.CONSTRUCTOR_TOKEN_PRIVATE_) &&
       opt_content) ||
      '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.html.TrustedResourceUrl#unwrap
   * @const {!Object}
   * @private
   */
  this.TRUSTED_RESOURCE_URL_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ =
      goog.html.TrustedResourceUrl.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_;
};


/**
 * @override
 * @const
 */
goog.html.TrustedResourceUrl.prototype.implementsGoogStringTypedString = true;


/**
 * Returns this TrustedResourceUrl's value as a string.
 *
 * IMPORTANT: In code where it is security relevant that an object's type is
 * indeed `TrustedResourceUrl`, use
 * `goog.html.TrustedResourceUrl.unwrap` instead of this method. If in
 * doubt, assume that it's security relevant. In particular, note that
 * goog.html functions which return a goog.html type do not guarantee that
 * the returned instance is of the right type. For example:
 *
 * <pre>
 * var fakeSafeHtml = new String('fake');
 * fakeSafeHtml.__proto__ = goog.html.SafeHtml.prototype;
 * var newSafeHtml = goog.html.SafeHtml.htmlEscape(fakeSafeHtml);
 * // newSafeHtml is just an alias for fakeSafeHtml, it's passed through by
 * // goog.html.SafeHtml.htmlEscape() as fakeSafeHtml instanceof
 * // goog.html.SafeHtml.
 * </pre>
 *
 * @see goog.html.TrustedResourceUrl#unwrap
 * @override
 */
goog.html.TrustedResourceUrl.prototype.getTypedStringValue = function() {
  return this.privateDoNotAccessOrElseTrustedResourceUrlWrappedValue_
      .toString();
};


/**
 * @override
 * @const
 */
goog.html.TrustedResourceUrl.prototype.implementsGoogI18nBidiDirectionalString =
    true;


/**
 * Returns this URLs directionality, which is always `LTR`.
 * @override
 */
goog.html.TrustedResourceUrl.prototype.getDirection = function() {
  return goog.i18n.bidi.Dir.LTR;
};


/**
 * Creates a new TrustedResourceUrl with params added to URL. Both search and
 * hash params can be specified.
 *
 * @param {string|?Object<string, *>|undefined} searchParams Search parameters
 *     to add to URL. See goog.html.TrustedResourceUrl.stringifyParams_ for
 *     exact format definition.
 * @param {(string|?Object<string, *>)=} opt_hashParams Hash parameters to add
 *     to URL. See goog.html.TrustedResourceUrl.stringifyParams_ for exact
 *     format definition.
 * @return {!goog.html.TrustedResourceUrl} New TrustedResourceUrl with params.
 */
goog.html.TrustedResourceUrl.prototype.cloneWithParams = function(
    searchParams, opt_hashParams) {
  var url = goog.html.TrustedResourceUrl.unwrap(this);
  var parts = goog.html.TrustedResourceUrl.URL_PARAM_PARSER_.exec(url);
  var urlBase = parts[1];
  var urlSearch = parts[2] || '';
  var urlHash = parts[3] || '';

  return goog.html.TrustedResourceUrl
      .createTrustedResourceUrlSecurityPrivateDoNotAccessOrElse(
          urlBase +
          goog.html.TrustedResourceUrl.stringifyParams_(
              '?', urlSearch, searchParams) +
          goog.html.TrustedResourceUrl.stringifyParams_(
              '#', urlHash, opt_hashParams));
};


if (goog.DEBUG) {
  /**
   * Returns a debug string-representation of this value.
   *
   * To obtain the actual string value wrapped in a TrustedResourceUrl, use
   * `goog.html.TrustedResourceUrl.unwrap`.
   *
   * @see goog.html.TrustedResourceUrl#unwrap
   * @override
   */
  goog.html.TrustedResourceUrl.prototype.toString = function() {
    return 'TrustedResourceUrl{' +
        this.privateDoNotAccessOrElseTrustedResourceUrlWrappedValue_ + '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed a
 * TrustedResourceUrl object, and returns its value.
 *
 * @param {!goog.html.TrustedResourceUrl} trustedResourceUrl The object to
 *     extract from.
 * @return {string} The trustedResourceUrl object's contained string, unless
 *     the run-time type check fails. In that case, `unwrap` returns an
 *     innocuous string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.html.TrustedResourceUrl.unwrap = function(trustedResourceUrl) {
  return goog.html.TrustedResourceUrl.unwrapTrustedScriptURL(trustedResourceUrl)
      .toString();
};


/**
 * Unwraps value as TrustedScriptURL if supported or as a string if not.
 * @param {!goog.html.TrustedResourceUrl} trustedResourceUrl
 * @return {!TrustedScriptURL|string}
 * @see goog.html.TrustedResourceUrl.unwrap
 */
goog.html.TrustedResourceUrl.unwrapTrustedScriptURL = function(
    trustedResourceUrl) {
  // Perform additional Run-time type-checking to ensure that
  // trustedResourceUrl is indeed an instance of the expected type.  This
  // provides some additional protection against security bugs due to
  // application code that disables type checks.
  // Specifically, the following checks are performed:
  // 1. The object is an instance of the expected type.
  // 2. The object is not an instance of a subclass.
  // 3. The object carries a type marker for the expected type. "Faking" an
  // object requires a reference to the type marker, which has names intended
  // to stand out in code reviews.
  if (trustedResourceUrl instanceof goog.html.TrustedResourceUrl &&
      trustedResourceUrl.constructor === goog.html.TrustedResourceUrl &&
      trustedResourceUrl
              .TRUSTED_RESOURCE_URL_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ ===
          goog.html.TrustedResourceUrl
              .TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_) {
    return trustedResourceUrl
        .privateDoNotAccessOrElseTrustedResourceUrlWrappedValue_;
  } else {
    goog.asserts.fail('expected object of type TrustedResourceUrl, got \'' +
        trustedResourceUrl + '\' of type ' + goog.typeOf(trustedResourceUrl));
    return 'type_error:TrustedResourceUrl';
  }
};


/**
 * Creates a TrustedResourceUrl from a format string and arguments.
 *
 * The arguments for interpolation into the format string map labels to values.
 * Values of type `goog.string.Const` are interpolated without modifcation.
 * Values of other types are cast to string and encoded with
 * encodeURIComponent.
 *
 * `%{<label>}` markers are used in the format string to indicate locations
 * to be interpolated with the valued mapped to the given label. `<label>`
 * must contain only alphanumeric and `_` characters.
 *
 * The format string must match goog.html.TrustedResourceUrl.BASE_URL_.
 *
 * Example usage:
 *
 *    var url = goog.html.TrustedResourceUrl.format(goog.string.Const.from(
 *        'https://www.google.com/search?q=%{query}'), {'query': searchTerm});
 *
 *    var url = goog.html.TrustedResourceUrl.format(goog.string.Const.from(
 *        '//www.youtube.com/v/%{videoId}?hl=en&fs=1%{autoplay}'), {
 *        'videoId': videoId,
 *        'autoplay': opt_autoplay ?
 *            goog.string.Const.from('&autoplay=1') : goog.string.Const.EMPTY
 *    });
 *
 * While this function can be used to create a TrustedResourceUrl from only
 * constants, fromConstant() and fromConstants() are generally preferable for
 * that purpose.
 *
 * @param {!goog.string.Const} format The format string.
 * @param {!Object<string, (string|number|!goog.string.Const)>} args Mapping
 *     of labels to values to be interpolated into the format string.
 *     goog.string.Const values are interpolated without encoding.
 * @return {!goog.html.TrustedResourceUrl}
 * @throws {!Error} On an invalid format string or if a label used in the
 *     the format string is not present in args.
 */
goog.html.TrustedResourceUrl.format = function(format, args) {
  var formatStr = goog.string.Const.unwrap(format);
  if (!goog.html.TrustedResourceUrl.BASE_URL_.test(formatStr)) {
    throw new Error('Invalid TrustedResourceUrl format: ' + formatStr);
  }
  var result = formatStr.replace(
      goog.html.TrustedResourceUrl.FORMAT_MARKER_, function(match, id) {
        if (!Object.prototype.hasOwnProperty.call(args, id)) {
          throw new Error(
              'Found marker, "' + id + '", in format string, "' + formatStr +
              '", but no valid label mapping found ' +
              'in args: ' + JSON.stringify(args));
        }
        var arg = args[id];
        if (arg instanceof goog.string.Const) {
          return goog.string.Const.unwrap(arg);
        } else {
          return encodeURIComponent(String(arg));
        }
      });
  return goog.html.TrustedResourceUrl
      .createTrustedResourceUrlSecurityPrivateDoNotAccessOrElse(result);
};


/**
 * @private @const {!RegExp}
 */
goog.html.TrustedResourceUrl.FORMAT_MARKER_ = /%{(\w+)}/g;


/**
 * The URL must be absolute, scheme-relative or path-absolute. So it must
 * start with:
 * - https:// followed by allowed origin characters.
 * - // followed by allowed origin characters.
 * - Any absolute or relative path.
 *
 * Based on
 * https://url.spec.whatwg.org/commit-snapshots/56b74ce7cca8883eab62e9a12666e2fac665d03d/#url-parsing
 * an initial / which is not followed by another / or \ will end up in the "path
 * state" and from there it can only go to "fragment state" and "query state".
 *
 * We don't enforce a well-formed domain name. So '.' or '1.2' are valid.
 * That's ok because the origin comes from a compile-time constant.
 *
 * A regular expression is used instead of goog.uri for several reasons:
 * - Strictness. E.g. we don't want any userinfo component and we don't
 *   want '/./, nor \' in the first path component.
 * - Small trusted base. goog.uri is generic and might need to change,
 *   reasoning about all the ways it can parse a URL now and in the future
 *   is error-prone.
 * - Code size. We expect many calls to .format(), many of which might
 *   not be using goog.uri.
 * - Simplicity. Using goog.uri would likely not result in simpler nor shorter
 *   code.
 * @private @const {!RegExp}
 */
goog.html.TrustedResourceUrl.BASE_URL_ = new RegExp(
    '^((https:)?//[0-9a-z.:[\\]-]+/'  // Origin.
        + '|/[^/\\\\]'                // Absolute path.
        + '|[^:/\\\\%]+/'             // Relative path.
        + '|[^:/\\\\%]*[?#]'          // Query string or fragment.
        + '|about:blank#'             // about:blank with fragment.
        + ')',
    'i');

/**
 * RegExp for splitting a URL into the base, search field, and hash field.
 *
 * @private @const {!RegExp}
 */
goog.html.TrustedResourceUrl.URL_PARAM_PARSER_ =
    /^([^?#]*)(\?[^#]*)?(#[\s\S]*)?/;


/**
 * Formats the URL same as TrustedResourceUrl.format and then adds extra URL
 * parameters.
 *
 * Example usage:
 *
 *     // Creates '//www.youtube.com/v/abc?autoplay=1' for videoId='abc' and
 *     // opt_autoplay=1. Creates '//www.youtube.com/v/abc' for videoId='abc'
 *     // and opt_autoplay=undefined.
 *     var url = goog.html.TrustedResourceUrl.formatWithParams(
 *         goog.string.Const.from('//www.youtube.com/v/%{videoId}'),
 *         {'videoId': videoId},
 *         {'autoplay': opt_autoplay});
 *
 * @param {!goog.string.Const} format The format string.
 * @param {!Object<string, (string|number|!goog.string.Const)>} args Mapping
 *     of labels to values to be interpolated into the format string.
 *     goog.string.Const values are interpolated without encoding.
 * @param {string|?Object<string, *>|undefined} searchParams Parameters to add
 *     to URL. See goog.html.TrustedResourceUrl.stringifyParams_ for exact
 *     format definition.
 * @param {(string|?Object<string, *>)=} opt_hashParams Hash parameters to add
 *     to URL. See goog.html.TrustedResourceUrl.stringifyParams_ for exact
 *     format definition.
 * @return {!goog.html.TrustedResourceUrl}
 * @throws {!Error} On an invalid format string or if a label used in the
 *     the format string is not present in args.
 */
goog.html.TrustedResourceUrl.formatWithParams = function(
    format, args, searchParams, opt_hashParams) {
  var url = goog.html.TrustedResourceUrl.format(format, args);
  return url.cloneWithParams(searchParams, opt_hashParams);
};


/**
 * Creates a TrustedResourceUrl object from a compile-time constant string.
 *
 * Compile-time constant strings are inherently program-controlled and hence
 * trusted.
 *
 * @param {!goog.string.Const} url A compile-time-constant string from which to
 *     create a TrustedResourceUrl.
 * @return {!goog.html.TrustedResourceUrl} A TrustedResourceUrl object
 *     initialized to `url`.
 */
goog.html.TrustedResourceUrl.fromConstant = function(url) {
  return goog.html.TrustedResourceUrl
      .createTrustedResourceUrlSecurityPrivateDoNotAccessOrElse(
          goog.string.Const.unwrap(url));
};


/**
 * Creates a TrustedResourceUrl object from a compile-time constant strings.
 *
 * Compile-time constant strings are inherently program-controlled and hence
 * trusted.
 *
 * @param {!Array<!goog.string.Const>} parts Compile-time-constant strings from
 *     which to create a TrustedResourceUrl.
 * @return {!goog.html.TrustedResourceUrl} A TrustedResourceUrl object
 *     initialized to concatenation of `parts`.
 */
goog.html.TrustedResourceUrl.fromConstants = function(parts) {
  var unwrapped = '';
  for (var i = 0; i < parts.length; i++) {
    unwrapped += goog.string.Const.unwrap(parts[i]);
  }
  return goog.html.TrustedResourceUrl
      .createTrustedResourceUrlSecurityPrivateDoNotAccessOrElse(unwrapped);
};

/**
 * Creates a TrustedResourceUrl object by generating a Blob from a SafeScript
 * object and then calling createObjectURL with that blob.
 *
 * SafeScript objects are trusted to contain executable JavaScript code.
 *
 * Caller must call goog.fs.url.revokeObjectUrl() on the unwrapped url to
 * release the underlying blob.
 *
 * Throws if browser doesn't support blob construction.
 *
 * @param {!goog.html.SafeScript} safeScript A script from which to create a
 *     TrustedResourceUrl.
 * @return {!goog.html.TrustedResourceUrl} A TrustedResourceUrl object
 *     initialized to a new blob URL.
 */
goog.html.TrustedResourceUrl.fromSafeScript = function(safeScript) {
  var blob = goog.fs.blob.getBlobWithProperties(
      [goog.html.SafeScript.unwrap(safeScript)], 'text/javascript');
  var url = goog.fs.url.createObjectUrl(blob);
  return goog.html.TrustedResourceUrl
      .createTrustedResourceUrlSecurityPrivateDoNotAccessOrElse(url);
};


/**
 * Type marker for the TrustedResourceUrl type, used to implement additional
 * run-time type checking.
 * @const {!Object}
 * @private
 */
goog.html.TrustedResourceUrl.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ = {};


/**
 * Package-internal utility method to create TrustedResourceUrl instances.
 *
 * @param {string} url The string to initialize the TrustedResourceUrl object
 *     with.
 * @return {!goog.html.TrustedResourceUrl} The initialized TrustedResourceUrl
 *     object.
 * @package
 */
goog.html.TrustedResourceUrl
    .createTrustedResourceUrlSecurityPrivateDoNotAccessOrElse = function(url) {
  var value = goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY ?
      goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY
          .createScriptURL(url) :
      url;
  return new goog.html.TrustedResourceUrl(
      goog.html.TrustedResourceUrl.CONSTRUCTOR_TOKEN_PRIVATE_, value);
};


/**
 * Stringifies the passed params to be used as either a search or hash field of
 * a URL.
 *
 * @param {string} prefix The prefix character for the given field ('?' or '#').
 * @param {string} currentString The existing field value (including the prefix
 *     character, if the field is present).
 * @param {string|?Object<string, *>|undefined} params The params to set or
 *     append to the field.
 * - If `undefined` or `null`, the field remains unchanged.
 * - If a string, then the string will be escaped and the field will be
 *   overwritten with that value.
 * - If an Object, that object is treated as a set of key-value pairs to be
 *   appended to the current field. Note that JavaScript doesn't guarantee the
 *   order of values in an object which might result in non-deterministic order
 *   of the parameters. However, browsers currently preserve the order. The
 *   rules for each entry:
 *   - If an array, it will be processed as if each entry were an additional
 *     parameter with exactly the same key, following the same logic below.
 *   - If `undefined` or `null`, it will be skipped.
 *   - Otherwise, it will be turned into a string, escaped, and appended.
 * @return {string}
 * @private
 */
goog.html.TrustedResourceUrl.stringifyParams_ = function(
    prefix, currentString, params) {
  if (params == null) {
    // Do not modify the field.
    return currentString;
  }
  if (typeof params === 'string') {
    // Set field to the passed string.
    return params ? prefix + encodeURIComponent(params) : '';
  }
  // Add on parameters to field from key-value object.
  for (var key in params) {
    var value = params[key];
    var outputValues = Array.isArray(value) ? value : [value];
    for (var i = 0; i < outputValues.length; i++) {
      var outputValue = outputValues[i];
      if (outputValue != null) {
        if (!currentString) {
          currentString = prefix;
        }
        currentString += (currentString.length > prefix.length ? '&' : '') +
            encodeURIComponent(key) + '=' +
            encodeURIComponent(String(outputValue));
      }
    }
  }
  return currentString;
};

/**
 * Token used to ensure that object is created only from this file. No code
 * outside of this file can access this token.
 * @private {!Object}
 * @const
 */
goog.html.TrustedResourceUrl.CONSTRUCTOR_TOKEN_PRIVATE_ = {};

//third_party/javascript/closure/string/internal.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview String functions called from Closure packages that couldn't
 * depend on each other. Outside Closure, use goog.string function which
 * delegate to these.
 * @visibility {//javascript/closure:__pkg__}
 * @visibility {//javascript/closure/bin/sizetests:__pkg__}
 * @visibility {//javascript/closure/dom:__pkg__}
 * @visibility {//javascript/closure/html:__pkg__}
 * @visibility {//javascript/closure/labs/useragent:__pkg__}
 */


goog.provide('goog.string.internal');


/**
 * Fast prefix-checker.
 * @param {string} str The string to check.
 * @param {string} prefix A string to look for at the start of `str`.
 * @return {boolean} True if `str` begins with `prefix`.
 * @see goog.string.startsWith
 */
goog.string.internal.startsWith = function(str, prefix) {
  return str.lastIndexOf(prefix, 0) == 0;
};


/**
 * Fast suffix-checker.
 * @param {string} str The string to check.
 * @param {string} suffix A string to look for at the end of `str`.
 * @return {boolean} True if `str` ends with `suffix`.
 * @see goog.string.endsWith
 */
goog.string.internal.endsWith = function(str, suffix) {
  const l = str.length - suffix.length;
  return l >= 0 && str.indexOf(suffix, l) == l;
};


/**
 * Case-insensitive prefix-checker.
 * @param {string} str The string to check.
 * @param {string} prefix  A string to look for at the end of `str`.
 * @return {boolean} True if `str` begins with `prefix` (ignoring
 *     case).
 * @see goog.string.caseInsensitiveStartsWith
 */
goog.string.internal.caseInsensitiveStartsWith = function(str, prefix) {
  return goog.string.internal.caseInsensitiveCompare(
             prefix, str.substr(0, prefix.length)) == 0;
};


/**
 * Case-insensitive suffix-checker.
 * @param {string} str The string to check.
 * @param {string} suffix A string to look for at the end of `str`.
 * @return {boolean} True if `str` ends with `suffix` (ignoring
 *     case).
 * @see goog.string.caseInsensitiveEndsWith
 */
goog.string.internal.caseInsensitiveEndsWith = function(str, suffix) {
  return (
      goog.string.internal.caseInsensitiveCompare(
          suffix, str.substr(str.length - suffix.length, suffix.length)) == 0);
};


/**
 * Case-insensitive equality checker.
 * @param {string} str1 First string to check.
 * @param {string} str2 Second string to check.
 * @return {boolean} True if `str1` and `str2` are the same string,
 *     ignoring case.
 * @see goog.string.caseInsensitiveEquals
 */
goog.string.internal.caseInsensitiveEquals = function(str1, str2) {
  return str1.toLowerCase() == str2.toLowerCase();
};


/**
 * Checks if a string is empty or contains only whitespaces.
 * @param {string} str The string to check.
 * @return {boolean} Whether `str` is empty or whitespace only.
 * @see goog.string.isEmptyOrWhitespace
 */
goog.string.internal.isEmptyOrWhitespace = function(str) {
  // testing length == 0 first is actually slower in all browsers (about the
  // same in Opera).
  // Since IE doesn't include non-breaking-space (0xa0) in their \s character
  // class (as required by section 7.2 of the ECMAScript spec), we explicitly
  // include it in the regexp to enforce consistent cross-browser behavior.
  return /^[\s\xa0]*$/.test(str);
};


/**
 * Trims white spaces to the left and right of a string.
 * @param {string} str The string to trim.
 * @return {string} A trimmed copy of `str`.
 */
goog.string.internal.trim =
    (goog.TRUSTED_SITE && String.prototype.trim) ? function(str) {
      return str.trim();
    } : function(str) {
      // Since IE doesn't include non-breaking-space (0xa0) in their \s
      // character class (as required by section 7.2 of the ECMAScript spec),
      // we explicitly include it in the regexp to enforce consistent
      // cross-browser behavior.
      // NOTE: We don't use String#replace because it might have side effects
      // causing this function to not compile to 0 bytes.
      return /^[\s\xa0]*([\s\S]*?)[\s\xa0]*$/.exec(str)[1];
    };


/**
 * A string comparator that ignores case.
 * -1 = str1 less than str2
 *  0 = str1 equals str2
 *  1 = str1 greater than str2
 *
 * @param {string} str1 The string to compare.
 * @param {string} str2 The string to compare `str1` to.
 * @return {number} The comparator result, as described above.
 * @see goog.string.caseInsensitiveCompare
 */
goog.string.internal.caseInsensitiveCompare = function(str1, str2) {
  const test1 = String(str1).toLowerCase();
  const test2 = String(str2).toLowerCase();

  if (test1 < test2) {
    return -1;
  } else if (test1 == test2) {
    return 0;
  } else {
    return 1;
  }
};


/**
 * Converts \n to <br>s or <br />s.
 * @param {string} str The string in which to convert newlines.
 * @param {boolean=} opt_xml Whether to use XML compatible tags.
 * @return {string} A copy of `str` with converted newlines.
 * @see goog.string.newLineToBr
 */
goog.string.internal.newLineToBr = function(str, opt_xml) {
  return str.replace(/(\r\n|\r|\n)/g, opt_xml ? '<br />' : '<br>');
};


/**
 * Escapes double quote '"' and single quote '\'' characters in addition to
 * '&', '<', and '>' so that a string can be included in an HTML tag attribute
 * value within double or single quotes.
 * @param {string} str string to be escaped.
 * @param {boolean=} opt_isLikelyToContainHtmlChars
 * @return {string} An escaped copy of `str`.
 * @see goog.string.htmlEscape
 */
goog.string.internal.htmlEscape = function(
    str, opt_isLikelyToContainHtmlChars) {
  if (opt_isLikelyToContainHtmlChars) {
    str = str.replace(goog.string.internal.AMP_RE_, '&amp;')
              .replace(goog.string.internal.LT_RE_, '&lt;')
              .replace(goog.string.internal.GT_RE_, '&gt;')
              .replace(goog.string.internal.QUOT_RE_, '&quot;')
              .replace(goog.string.internal.SINGLE_QUOTE_RE_, '&#39;')
              .replace(goog.string.internal.NULL_RE_, '&#0;');
    return str;

  } else {
    // quick test helps in the case when there are no chars to replace, in
    // worst case this makes barely a difference to the time taken
    if (!goog.string.internal.ALL_RE_.test(str)) return str;

    // str.indexOf is faster than regex.test in this case
    if (str.indexOf('&') != -1) {
      str = str.replace(goog.string.internal.AMP_RE_, '&amp;');
    }
    if (str.indexOf('<') != -1) {
      str = str.replace(goog.string.internal.LT_RE_, '&lt;');
    }
    if (str.indexOf('>') != -1) {
      str = str.replace(goog.string.internal.GT_RE_, '&gt;');
    }
    if (str.indexOf('"') != -1) {
      str = str.replace(goog.string.internal.QUOT_RE_, '&quot;');
    }
    if (str.indexOf('\'') != -1) {
      str = str.replace(goog.string.internal.SINGLE_QUOTE_RE_, '&#39;');
    }
    if (str.indexOf('\x00') != -1) {
      str = str.replace(goog.string.internal.NULL_RE_, '&#0;');
    }
    return str;
  }
};


/**
 * Regular expression that matches an ampersand, for use in escaping.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.AMP_RE_ = /&/g;


/**
 * Regular expression that matches a less than sign, for use in escaping.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.LT_RE_ = /</g;


/**
 * Regular expression that matches a greater than sign, for use in escaping.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.GT_RE_ = />/g;


/**
 * Regular expression that matches a double quote, for use in escaping.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.QUOT_RE_ = /"/g;


/**
 * Regular expression that matches a single quote, for use in escaping.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.SINGLE_QUOTE_RE_ = /'/g;


/**
 * Regular expression that matches null character, for use in escaping.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.NULL_RE_ = /\x00/g;


/**
 * Regular expression that matches any character that needs to be escaped.
 * @const {!RegExp}
 * @private
 */
goog.string.internal.ALL_RE_ = /[\x00&<>"']/;


/**
 * Do escaping of whitespace to preserve spatial formatting. We use character
 * entity #160 to make it safer for xml.
 * @param {string} str The string in which to escape whitespace.
 * @param {boolean=} opt_xml Whether to use XML compatible tags.
 * @return {string} An escaped copy of `str`.
 * @see goog.string.whitespaceEscape
 */
goog.string.internal.whitespaceEscape = function(str, opt_xml) {
  // This doesn't use goog.string.preserveSpaces for backwards compatibility.
  return goog.string.internal.newLineToBr(
      str.replace(/  /g, ' &#160;'), opt_xml);
};


/**
 * Determines whether a string contains a substring.
 * @param {string} str The string to search.
 * @param {string} subString The substring to search for.
 * @return {boolean} Whether `str` contains `subString`.
 * @see goog.string.contains
 */
goog.string.internal.contains = function(str, subString) {
  return str.indexOf(subString) != -1;
};


/**
 * Determines whether a string contains a substring, ignoring case.
 * @param {string} str The string to search.
 * @param {string} subString The substring to search for.
 * @return {boolean} Whether `str` contains `subString`.
 * @see goog.string.caseInsensitiveContains
 */
goog.string.internal.caseInsensitiveContains = function(str, subString) {
  return goog.string.internal.contains(
      str.toLowerCase(), subString.toLowerCase());
};


/**
 * Compares two version numbers.
 *
 * @param {string|number} version1 Version of first item.
 * @param {string|number} version2 Version of second item.
 *
 * @return {number}  1 if `version1` is higher.
 *                   0 if arguments are equal.
 *                  -1 if `version2` is higher.
 * @see goog.string.compareVersions
 */
goog.string.internal.compareVersions = function(version1, version2) {
  let order = 0;
  // Trim leading and trailing whitespace and split the versions into
  // subversions.
  const v1Subs = goog.string.internal.trim(String(version1)).split('.');
  const v2Subs = goog.string.internal.trim(String(version2)).split('.');
  const subCount = Math.max(v1Subs.length, v2Subs.length);

  // Iterate over the subversions, as long as they appear to be equivalent.
  for (let subIdx = 0; order == 0 && subIdx < subCount; subIdx++) {
    let v1Sub = v1Subs[subIdx] || '';
    let v2Sub = v2Subs[subIdx] || '';

    do {
      // Split the subversions into pairs of numbers and qualifiers (like 'b').
      // Two different RegExp objects are use to make it clear the code
      // is side-effect free
      const v1Comp = /(\d*)(\D*)(.*)/.exec(v1Sub) || ['', '', '', ''];
      const v2Comp = /(\d*)(\D*)(.*)/.exec(v2Sub) || ['', '', '', ''];
      // Break if there are no more matches.
      if (v1Comp[0].length == 0 && v2Comp[0].length == 0) {
        break;
      }

      // Parse the numeric part of the subversion. A missing number is
      // equivalent to 0.
      const v1CompNum = v1Comp[1].length == 0 ? 0 : parseInt(v1Comp[1], 10);
      const v2CompNum = v2Comp[1].length == 0 ? 0 : parseInt(v2Comp[1], 10);

      // Compare the subversion components. The number has the highest
      // precedence. Next, if the numbers are equal, a subversion without any
      // qualifier is always higher than a subversion with any qualifier. Next,
      // the qualifiers are compared as strings.
      order = goog.string.internal.compareElements_(v1CompNum, v2CompNum) ||
          goog.string.internal.compareElements_(
              v1Comp[2].length == 0, v2Comp[2].length == 0) ||
          goog.string.internal.compareElements_(v1Comp[2], v2Comp[2]);
      // Stop as soon as an inequality is discovered.

      v1Sub = v1Comp[3];
      v2Sub = v2Comp[3];
    } while (order == 0);
  }

  return order;
};


/**
 * Compares elements of a version number.
 *
 * @param {string|number|boolean} left An element from a version number.
 * @param {string|number|boolean} right An element from a version number.
 *
 * @return {number}  1 if `left` is higher.
 *                   0 if arguments are equal.
 *                  -1 if `right` is higher.
 * @private
 */
goog.string.internal.compareElements_ = function(left, right) {
  if (left < right) {
    return -1;
  } else if (left > right) {
    return 1;
  }
  return 0;
};

//third_party/javascript/closure/html/safeurl.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview The SafeUrl type and its builders.
 *
 * TODO(xtof): Link to document stating type contract.
 */

goog.provide('goog.html.SafeUrl');

goog.require('goog.asserts');
goog.require('goog.fs.url');
goog.require('goog.html.TrustedResourceUrl');
goog.require('goog.i18n.bidi.Dir');
goog.require('goog.i18n.bidi.DirectionalString');
goog.require('goog.string.Const');
goog.require('goog.string.TypedString');
goog.require('goog.string.internal');



/**
 * A string that is safe to use in URL context in DOM APIs and HTML documents.
 *
 * A SafeUrl is a string-like object that carries the security type contract
 * that its value as a string will not cause untrusted script execution
 * when evaluated as a hyperlink URL in a browser.
 *
 * Values of this type are guaranteed to be safe to use in URL/hyperlink
 * contexts, such as assignment to URL-valued DOM properties, in the sense that
 * the use will not result in a Cross-Site-Scripting vulnerability. Similarly,
 * SafeUrls can be interpolated into the URL context of an HTML template (e.g.,
 * inside a href attribute). However, appropriate HTML-escaping must still be
 * applied.
 *
 * Note that, as documented in `goog.html.SafeUrl.unwrap`, this type's
 * contract does not guarantee that instances are safe to interpolate into HTML
 * without appropriate escaping.
 *
 * Note also that this type's contract does not imply any guarantees regarding
 * the resource the URL refers to.  In particular, SafeUrls are <b>not</b>
 * safe to use in a context where the referred-to resource is interpreted as
 * trusted code, e.g., as the src of a script tag.
 *
 * Instances of this type must be created via the factory methods
 * (`goog.html.SafeUrl.fromConstant`, `goog.html.SafeUrl.sanitize`),
 * etc and not by invoking its constructor. The constructor is organized in a
 * way that only methods from that file can call it and initialize with
 * non-empty values. Anyone else calling constructor will get default instance
 * with empty value.
 *
 * @see goog.html.SafeUrl#fromConstant
 * @see goog.html.SafeUrl#from
 * @see goog.html.SafeUrl#sanitize
 * @constructor
 * @final
 * @struct
 * @implements {goog.i18n.bidi.DirectionalString}
 * @implements {goog.string.TypedString}
 * @param {!Object=} opt_token package-internal implementation detail.
 * @param {string=} opt_content package-internal implementation detail.
 */
goog.html.SafeUrl = function(opt_token, opt_content) {
  /**
   * The contained value of this SafeUrl.  The field has a purposely ugly
   * name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @private {string}
   */
  this.privateDoNotAccessOrElseSafeUrlWrappedValue_ =
      ((opt_token === goog.html.SafeUrl.CONSTRUCTOR_TOKEN_PRIVATE_) &&
       opt_content) ||
      '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.html.SafeUrl#unwrap
   * @const {!Object}
   * @private
   */
  this.SAFE_URL_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ =
      goog.html.SafeUrl.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_;
};


/**
 * The innocuous string generated by goog.html.SafeUrl.sanitize when passed
 * an unsafe URL.
 *
 * about:invalid is registered in
 * http://www.w3.org/TR/css3-values/#about-invalid.
 * http://tools.ietf.org/html/rfc6694#section-2.2.1 permits about URLs to
 * contain a fragment, which is not to be considered when determining if an
 * about URL is well-known.
 *
 * Using about:invalid seems preferable to using a fixed data URL, since
 * browsers might choose to not report CSP violations on it, as legitimate
 * CSS function calls to attr() can result in this URL being produced. It is
 * also a standard URL which matches exactly the semantics we need:
 * "The about:invalid URI references a non-existent document with a generic
 * error condition. It can be used when a URI is necessary, but the default
 * value shouldn't be resolveable as any type of document".
 *
 * @const {string}
 */
goog.html.SafeUrl.INNOCUOUS_STRING = 'about:invalid#zClosurez';


/**
 * @override
 * @const
 */
goog.html.SafeUrl.prototype.implementsGoogStringTypedString = true;


/**
 * Returns this SafeUrl's value as a string.
 *
 * IMPORTANT: In code where it is security relevant that an object's type is
 * indeed `SafeUrl`, use `goog.html.SafeUrl.unwrap` instead of this
 * method. If in doubt, assume that it's security relevant. In particular, note
 * that goog.html functions which return a goog.html type do not guarantee that
 * the returned instance is of the right type.
 *
 * IMPORTANT: The guarantees of the SafeUrl type contract only extend to the
 * behavior of browsers when interpreting URLs. Values of SafeUrl objects MUST
 * be appropriately escaped before embedding in a HTML document. Note that the
 * required escaping is context-sensitive (e.g. a different escaping is
 * required for embedding a URL in a style property within a style
 * attribute, as opposed to embedding in a href attribute).
 *
 * @see goog.html.SafeUrl#unwrap
 * @override
 */
goog.html.SafeUrl.prototype.getTypedStringValue = function() {
  return this.privateDoNotAccessOrElseSafeUrlWrappedValue_.toString();
};


/**
 * @override
 * @const
 */
goog.html.SafeUrl.prototype.implementsGoogI18nBidiDirectionalString = true;


/**
 * Returns this URLs directionality, which is always `LTR`.
 * @override
 */
goog.html.SafeUrl.prototype.getDirection = function() {
  return goog.i18n.bidi.Dir.LTR;
};


if (goog.DEBUG) {
  /**
   * Returns a debug string-representation of this value.
   *
   * To obtain the actual string value wrapped in a SafeUrl, use
   * `goog.html.SafeUrl.unwrap`.
   *
   * @see goog.html.SafeUrl#unwrap
   * @override
   */
  goog.html.SafeUrl.prototype.toString = function() {
    return 'SafeUrl{' + this.privateDoNotAccessOrElseSafeUrlWrappedValue_ + '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed a SafeUrl
 * object, and returns its value.
 *
 * IMPORTANT: The guarantees of the SafeUrl type contract only extend to the
 * behavior of  browsers when interpreting URLs. Values of SafeUrl objects MUST
 * be appropriately escaped before embedding in a HTML document. Note that the
 * required escaping is context-sensitive (e.g. a different escaping is
 * required for embedding a URL in a style property within a style
 * attribute, as opposed to embedding in a href attribute).
 *
 * @param {!goog.html.SafeUrl} safeUrl The object to extract from.
 * @return {string} The SafeUrl object's contained string, unless the run-time
 *     type check fails. In that case, `unwrap` returns an innocuous
 *     string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.html.SafeUrl.unwrap = function(safeUrl) {
  // Perform additional Run-time type-checking to ensure that safeUrl is indeed
  // an instance of the expected type.  This provides some additional protection
  // against security bugs due to application code that disables type checks.
  // Specifically, the following checks are performed:
  // 1. The object is an instance of the expected type.
  // 2. The object is not an instance of a subclass.
  // 3. The object carries a type marker for the expected type. "Faking" an
  // object requires a reference to the type marker, which has names intended
  // to stand out in code reviews.
  if (safeUrl instanceof goog.html.SafeUrl &&
      safeUrl.constructor === goog.html.SafeUrl &&
      safeUrl.SAFE_URL_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ ===
          goog.html.SafeUrl.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_) {
    return safeUrl.privateDoNotAccessOrElseSafeUrlWrappedValue_;
  } else {
    goog.asserts.fail('expected object of type SafeUrl, got \'' +
        safeUrl + '\' of type ' + goog.typeOf(safeUrl));
    return 'type_error:SafeUrl';
  }
};


/**
 * Creates a SafeUrl object from a compile-time constant string.
 *
 * Compile-time constant strings are inherently program-controlled and hence
 * trusted.
 *
 * @param {!goog.string.Const} url A compile-time-constant string from which to
 *         create a SafeUrl.
 * @return {!goog.html.SafeUrl} A SafeUrl object initialized to `url`.
 */
goog.html.SafeUrl.fromConstant = function(url) {
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      goog.string.Const.unwrap(url));
};


/**
 * A pattern that matches Blob or data types that can have SafeUrls created
 * from URL.createObjectURL(blob) or via a data: URI.
 *
 * This has some parameter support (most notably, we haven't implemented the
 * more complex parts like %-encoded characters or non-alphanumerical ones for
 * simplicity's sake). The specs are fairly complex, and they don't
 * always match Chrome's behavior: we settled on a subset where we're confident
 * all parties involved agree.
 *
 * The spec is available at https://mimesniff.spec.whatwg.org/ (and see
 * https://tools.ietf.org/html/rfc2397 for data: urls, which override some of
 * it).
 * @const
 * @private
 */
goog.html.SAFE_MIME_TYPE_PATTERN_ = new RegExp(
    // Note: Due to content-sniffing concerns, only add MIME types for
    // media formats.
    '^(?:audio/(?:3gpp2|3gpp|aac|L16|midi|mp3|mp4|mpeg|oga|ogg|opus|x-m4a|x-matroska|x-wav|wav|webm)|' +
        'font/\\w+|' +
        'image/(?:bmp|gif|jpeg|jpg|png|tiff|webp|x-icon)|' +
        // TODO(b/68188949): Due to content-sniffing concerns, text/csv should
        // be removed from the whitelist.
        'text/csv|' +
        'video/(?:mpeg|mp4|ogg|webm|quicktime|x-matroska))' +
        '(?:;\\w+=(?:\\w+|"[\\w;,= ]+"))*$',  // MIME type parameters
    'i');


/**
 * @param {string} mimeType The MIME type to check if safe.
 * @return {boolean} True if the MIME type is safe and creating a Blob via
 *   `SafeUrl.fromBlob()` with that type will not fail due to the type. False
 *   otherwise.
 */
goog.html.SafeUrl.isSafeMimeType = function(mimeType) {
  return goog.html.SAFE_MIME_TYPE_PATTERN_.test(mimeType);
};


/**
 * Creates a SafeUrl wrapping a blob URL for the given `blob`.
 *
 * The blob URL is created with `URL.createObjectURL`. If the MIME type
 * for `blob` is not of a known safe audio, image or video MIME type,
 * then the SafeUrl will wrap {@link #INNOCUOUS_STRING}.
 *
 * @see http://www.w3.org/TR/FileAPI/#url
 * @param {!Blob} blob
 * @return {!goog.html.SafeUrl} The blob URL, or an innocuous string wrapped
 *   as a SafeUrl.
 */
goog.html.SafeUrl.fromBlob = function(blob) {
  var url = goog.html.SafeUrl.isSafeMimeType(blob.type) ?
      goog.fs.url.createObjectUrl(blob) :
      goog.html.SafeUrl.INNOCUOUS_STRING;
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(url);
};


/**
 * Creates a SafeUrl wrapping a blob URL created for a MediaSource.
 * @param {!MediaSource} mediaSource
 * @return {!goog.html.SafeUrl} The blob URL.
 */
goog.html.SafeUrl.fromMediaSource = function(mediaSource) {
  goog.asserts.assert(
      'MediaSource' in goog.global, 'No support for MediaSource');
  const url = mediaSource instanceof MediaSource ?
      goog.fs.url.createObjectUrl(mediaSource) :
      goog.html.SafeUrl.INNOCUOUS_STRING;
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(url);
};


/**
 * Matches a base-64 data URL, with the first match group being the MIME type.
 * @const
 * @private
 */
goog.html.DATA_URL_PATTERN_ = /^data:(.*);base64,[a-z0-9+\/]+=*$/i;


/**
 * Creates a SafeUrl wrapping a data: URL, after validating it matches a
 * known-safe media MIME type.
 *
 * @param {string} dataUrl A valid base64 data URL with one of the whitelisted
 *     media MIME types.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromDataUrl = function(dataUrl) {
  // RFC4648 suggest to ignore CRLF in base64 encoding.
  // See https://tools.ietf.org/html/rfc4648.
  // Remove the CR (%0D) and LF (%0A) from the dataUrl.
  var filteredDataUrl = dataUrl.replace(/(%0A|%0D)/g, '');
  var match = filteredDataUrl.match(goog.html.DATA_URL_PATTERN_);
  // Note: The only risk of XSS here is if the `data:` URL results in a
  // same-origin document. In which case content-sniffing might cause the
  // browser to interpret the contents as html.
  // All modern browsers consider `data:` URL documents to have unique empty
  // origins. Only Firefox for versions prior to v57 behaves differently:
  // https://blog.mozilla.org/security/2017/10/04/treating-data-urls-unique-origins-firefox-57/
  // Older versions of IE don't understand `data:` urls, so it is not an issue.
  var valid = match && goog.html.SafeUrl.isSafeMimeType(match[1]);
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      valid ? filteredDataUrl : goog.html.SafeUrl.INNOCUOUS_STRING);
};


/**
 * Creates a SafeUrl wrapping a tel: URL.
 *
 * @param {string} telUrl A tel URL.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromTelUrl = function(telUrl) {
  // There's a risk that a tel: URL could immediately place a call once
  // clicked, without requiring user confirmation. For that reason it is
  // handled in this separate function.
  if (!goog.string.internal.caseInsensitiveStartsWith(telUrl, 'tel:')) {
    telUrl = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      telUrl);
};


/**
 * Matches a sip/sips URL. We only allow urls that consist of an email address.
 * The characters '?' and '#' are not allowed in the local part of the email
 * address.
 * @const
 * @private
 */
goog.html.SIP_URL_PATTERN_ = new RegExp(
    '^sip[s]?:[+a-z0-9_.!$%&\'*\\/=^`{|}~-]+@([a-z0-9-]+\\.)+[a-z0-9]{2,63}$',
    'i');


/**
 * Creates a SafeUrl wrapping a sip: URL. We only allow urls that consist of an
 * email address. The characters '?' and '#' are not allowed in the local part
 * of the email address.
 *
 * @param {string} sipUrl A sip URL.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromSipUrl = function(sipUrl) {
  if (!goog.html.SIP_URL_PATTERN_.test(decodeURIComponent(sipUrl))) {
    sipUrl = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      sipUrl);
};


/**
 * Creates a SafeUrl wrapping a fb-messenger://share URL.
 *
 * @param {string} facebookMessengerUrl A facebook messenger URL.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromFacebookMessengerUrl = function(facebookMessengerUrl) {
  if (!goog.string.internal.caseInsensitiveStartsWith(
          facebookMessengerUrl, 'fb-messenger://share')) {
    facebookMessengerUrl = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      facebookMessengerUrl);
};

/**
 * Creates a SafeUrl wrapping a whatsapp://send URL.
 *
 * @param {string} whatsAppUrl A WhatsApp URL.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromWhatsAppUrl = function(whatsAppUrl) {
  if (!goog.string.internal.caseInsensitiveStartsWith(
          whatsAppUrl, 'whatsapp://send')) {
    whatsAppUrl = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      whatsAppUrl);
};

/**
 * Creates a SafeUrl wrapping a sms: URL.
 *
 * @param {string} smsUrl A sms URL.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromSmsUrl = function(smsUrl) {
  if (!goog.string.internal.caseInsensitiveStartsWith(smsUrl, 'sms:') ||
      !goog.html.SafeUrl.isSmsUrlBodyValid_(smsUrl)) {
    smsUrl = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      smsUrl);
};


/**
 * Validates SMS URL `body` parameter, which is optional and should appear at
 * most once and should be percent-encoded if present. Rejects many malformed
 * bodies, but may spuriously reject some URLs and does not reject all malformed
 * sms: URLs.
 *
 * @param {string} smsUrl A sms URL.
 * @return {boolean} Whether SMS URL has a valid `body` parameter if it exists.
 * @private
 */
goog.html.SafeUrl.isSmsUrlBodyValid_ = function(smsUrl) {
  var hash = smsUrl.indexOf('#');
  if (hash > 0) {
    smsUrl = smsUrl.substring(0, hash);
  }
  var bodyParams = smsUrl.match(/[?&]body=/gi);
  // "body" param is optional
  if (!bodyParams) {
    return true;
  }
  // "body" MUST only appear once
  if (bodyParams.length > 1) {
    return false;
  }
  // Get the encoded `body` parameter value.
  var bodyValue = smsUrl.match(/[?&]body=([^&]*)/)[1];
  if (!bodyValue) {
    return true;
  }
  try {
    decodeURIComponent(bodyValue);
  } catch (error) {
    return false;
  }
  return /^(?:[a-z0-9\-_.~]|%[0-9a-f]{2})+$/i.test(bodyValue);
};


/**
 * Creates a SafeUrl wrapping a ssh: URL.
 *
 * @param {string} sshUrl A ssh URL.
 * @return {!goog.html.SafeUrl} A matching safe URL, or {@link INNOCUOUS_STRING}
 *     wrapped as a SafeUrl if it does not pass.
 */
goog.html.SafeUrl.fromSshUrl = function(sshUrl) {
  if (!goog.string.internal.caseInsensitiveStartsWith(sshUrl, 'ssh://')) {
    sshUrl = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      sshUrl);
};

/**
 * Sanitizes a Chrome extension URL to SafeUrl, given a compile-time-constant
 * extension identifier. Can also be restricted to chrome extensions.
 *
 * @param {string} url The url to sanitize. Should start with the extension
 *     scheme and the extension identifier.
 * @param {!goog.string.Const|!Array<!goog.string.Const>} extensionId The
 *     extension id to accept, as a compile-time constant or an array of those.
 *
 * @return {!goog.html.SafeUrl} Either `url` if it's deemed safe, or
 *     `INNOCUOUS_STRING` if it's not.
 */
goog.html.SafeUrl.sanitizeChromeExtensionUrl = function(url, extensionId) {
  return goog.html.SafeUrl.sanitizeExtensionUrl_(
      /^chrome-extension:\/\/([^\/]+)\//, url, extensionId);
};

/**
 * Sanitizes a Firefox extension URL to SafeUrl, given a compile-time-constant
 * extension identifier. Can also be restricted to chrome extensions.
 *
 * @param {string} url The url to sanitize. Should start with the extension
 *     scheme and the extension identifier.
 * @param {!goog.string.Const|!Array<!goog.string.Const>} extensionId The
 *     extension id to accept, as a compile-time constant or an array of those.
 *
 * @return {!goog.html.SafeUrl} Either `url` if it's deemed safe, or
 *     `INNOCUOUS_STRING` if it's not.
 */
goog.html.SafeUrl.sanitizeFirefoxExtensionUrl = function(url, extensionId) {
  return goog.html.SafeUrl.sanitizeExtensionUrl_(
      /^moz-extension:\/\/([^\/]+)\//, url, extensionId);
};

/**
 * Sanitizes a Edge extension URL to SafeUrl, given a compile-time-constant
 * extension identifier. Can also be restricted to chrome extensions.
 *
 * @param {string} url The url to sanitize. Should start with the extension
 *     scheme and the extension identifier.
 * @param {!goog.string.Const|!Array<!goog.string.Const>} extensionId The
 *     extension id to accept, as a compile-time constant or an array of those.
 *
 * @return {!goog.html.SafeUrl} Either `url` if it's deemed safe, or
 *     `INNOCUOUS_STRING` if it's not.
 */
goog.html.SafeUrl.sanitizeEdgeExtensionUrl = function(url, extensionId) {
  return goog.html.SafeUrl.sanitizeExtensionUrl_(
      /^ms-browser-extension:\/\/([^\/]+)\//, url, extensionId);
};

/**
 * Private helper for converting extension URLs to SafeUrl, given the scheme for
 * that particular extension type. Use the sanitizeFirefoxExtensionUrl,
 * sanitizeChromeExtensionUrl or sanitizeEdgeExtensionUrl unless you're building
 * new helpers.
 *
 * @private
 * @param {!RegExp} scheme The scheme to accept as a RegExp extracting the
 *     extension identifier.
 * @param {string} url The url to sanitize. Should start with the extension
 *     scheme and the extension identifier.
 * @param {!goog.string.Const|!Array<!goog.string.Const>} extensionId The
 *     extension id to accept, as a compile-time constant or an array of those.
 *
 * @return {!goog.html.SafeUrl} Either `url` if it's deemed safe, or
 *     `INNOCUOUS_STRING` if it's not.
 */
goog.html.SafeUrl.sanitizeExtensionUrl_ = function(scheme, url, extensionId) {
  var matches = scheme.exec(url);
  if (!matches) {
    url = goog.html.SafeUrl.INNOCUOUS_STRING;
  } else {
    var extractedExtensionId = matches[1];
    var acceptedExtensionIds;
    if (extensionId instanceof goog.string.Const) {
      acceptedExtensionIds = [goog.string.Const.unwrap(extensionId)];
    } else {
      acceptedExtensionIds = extensionId.map(function unwrap(x) {
        return goog.string.Const.unwrap(x);
      });
    }
    if (acceptedExtensionIds.indexOf(extractedExtensionId) == -1) {
      url = goog.html.SafeUrl.INNOCUOUS_STRING;
    }
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(url);
};


/**
 * Creates a SafeUrl from TrustedResourceUrl. This is safe because
 * TrustedResourceUrl is more tightly restricted than SafeUrl.
 *
 * @param {!goog.html.TrustedResourceUrl} trustedResourceUrl
 * @return {!goog.html.SafeUrl}
 */
goog.html.SafeUrl.fromTrustedResourceUrl = function(trustedResourceUrl) {
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
      goog.html.TrustedResourceUrl.unwrap(trustedResourceUrl));
};


/**
 * A pattern that recognizes a commonly useful subset of URLs that satisfy
 * the SafeUrl contract.
 *
 * This regular expression matches a subset of URLs that will not cause script
 * execution if used in URL context within a HTML document. Specifically, this
 * regular expression matches if (comment from here on and regex copied from
 * Soy's EscapingConventions):
 * (1) Either a protocol in a whitelist (http, https, mailto or ftp).
 * (2) or no protocol.  A protocol must be followed by a colon. The below
 *     allows that by allowing colons only after one of the characters [/?#].
 *     A colon after a hash (#) must be in the fragment.
 *     Otherwise, a colon after a (?) must be in a query.
 *     Otherwise, a colon after a single solidus (/) must be in a path.
 *     Otherwise, a colon after a double solidus (//) must be in the authority
 *     (before port).
 *
 * @private
 * @const {!RegExp}
 */
goog.html.SAFE_URL_PATTERN_ =
    /^(?:(?:https?|mailto|ftp):|[^:/?#]*(?:[/?#]|$))/i;

/**
 * Public version of goog.html.SAFE_URL_PATTERN_. Updating
 * goog.html.SAFE_URL_PATTERN_ doesn't seem to be backward compatible.
 * Namespace is also changed to goog.html.SafeUrl so it can be imported using
 * goog.require('goog.dom.SafeUrl').
 *
 * TODO(bangert): Remove SAFE_URL_PATTERN_
 * @const {!RegExp}
 */
goog.html.SafeUrl.SAFE_URL_PATTERN = goog.html.SAFE_URL_PATTERN_;


/**
 * Creates a SafeUrl object from `url`. If `url` is a
 * goog.html.SafeUrl then it is simply returned. Otherwise the input string is
 * validated to match a pattern of commonly used safe URLs.
 *
 * `url` may be a URL with the http, https, mailto or ftp scheme,
 * or a relative URL (i.e., a URL without a scheme; specifically, a
 * scheme-relative, absolute-path-relative, or path-relative URL).
 *
 * @see http://url.spec.whatwg.org/#concept-relative-url
 * @param {string|!goog.string.TypedString} url The URL to validate.
 * @return {!goog.html.SafeUrl} The validated URL, wrapped as a SafeUrl.
 */
goog.html.SafeUrl.sanitize = function(url) {
  if (url instanceof goog.html.SafeUrl) {
    return url;
  } else if (typeof url == 'object' && url.implementsGoogStringTypedString) {
    url = /** @type {!goog.string.TypedString} */ (url).getTypedStringValue();
  } else {
    url = String(url);
  }
  if (!goog.html.SAFE_URL_PATTERN_.test(url)) {
    url = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(url);
};

/**
 * Creates a SafeUrl object from `url`. If `url` is a
 * goog.html.SafeUrl then it is simply returned. Otherwise the input string is
 * validated to match a pattern of commonly used safe URLs.
 *
 * `url` may be a URL with the http, https, mailto or ftp scheme,
 * or a relative URL (i.e., a URL without a scheme; specifically, a
 * scheme-relative, absolute-path-relative, or path-relative URL).
 *
 * This function asserts (using goog.asserts) that the URL matches this pattern.
 * If it does not, in addition to failing the assert, an innocous URL will be
 * returned.
 *
 * @see http://url.spec.whatwg.org/#concept-relative-url
 * @param {string|!goog.string.TypedString} url The URL to validate.
 * @param {boolean=} opt_allowDataUrl Whether to allow valid data: URLs.
 * @return {!goog.html.SafeUrl} The validated URL, wrapped as a SafeUrl.
 */
goog.html.SafeUrl.sanitizeAssertUnchanged = function(url, opt_allowDataUrl) {
  if (url instanceof goog.html.SafeUrl) {
    return url;
  } else if (typeof url == 'object' && url.implementsGoogStringTypedString) {
    url = /** @type {!goog.string.TypedString} */ (url).getTypedStringValue();
  } else {
    url = String(url);
  }
  if (opt_allowDataUrl && /^data:/i.test(url)) {
    var safeUrl = goog.html.SafeUrl.fromDataUrl(url);
    if (safeUrl.getTypedStringValue() == url) {
      return safeUrl;
    }
  }
  if (!goog.asserts.assert(
          goog.html.SAFE_URL_PATTERN_.test(url),
          '%s does not match the safe URL pattern', url)) {
    url = goog.html.SafeUrl.INNOCUOUS_STRING;
  }
  return goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(url);
};



/**
 * Type marker for the SafeUrl type, used to implement additional run-time
 * type checking.
 * @const {!Object}
 * @private
 */
goog.html.SafeUrl.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ = {};


/**
 * Package-internal utility method to create SafeUrl instances.
 *
 * @param {string} url The string to initialize the SafeUrl object with.
 * @return {!goog.html.SafeUrl} The initialized SafeUrl object.
 * @package
 */
goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse = function(
    url) {
  return new goog.html.SafeUrl(
      goog.html.SafeUrl.CONSTRUCTOR_TOKEN_PRIVATE_, url);
};


/**
 * A SafeUrl corresponding to the special about:blank url.
 * @const {!goog.html.SafeUrl}
 */
goog.html.SafeUrl.ABOUT_BLANK =
    goog.html.SafeUrl.createSafeUrlSecurityPrivateDoNotAccessOrElse(
        'about:blank');

/**
 * Token used to ensure that object is created only from this file. No code
 * outside of this file can access this token.
 * @private {!Object}
 * @const
 */
goog.html.SafeUrl.CONSTRUCTOR_TOKEN_PRIVATE_ = {};

//third_party/javascript/closure/html/safestyle.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview The SafeStyle type and its builders.
 *
 * TODO(xtof): Link to document stating type contract.
 */

goog.provide('goog.html.SafeStyle');

goog.require('goog.array');
goog.require('goog.asserts');
goog.require('goog.html.SafeUrl');
goog.require('goog.string.Const');
goog.require('goog.string.TypedString');
goog.require('goog.string.internal');



/**
 * A string-like object which represents a sequence of CSS declarations
 * ({@code propertyName1: propertyvalue1; propertyName2: propertyValue2; ...})
 * and that carries the security type contract that its value, as a string,
 * will not cause untrusted script execution (XSS) when evaluated as CSS in a
 * browser.
 *
 * Instances of this type must be created via the factory methods
 * (`goog.html.SafeStyle.create` or
 * `goog.html.SafeStyle.fromConstant`) and not by invoking its
 * constructor. The constructor intentionally takes no parameters and the type
 * is immutable; hence only a default instance corresponding to the empty string
 * can be obtained via constructor invocation.
 *
 * SafeStyle's string representation can safely be:
 * <ul>
 *   <li>Interpolated as the content of a *quoted* HTML style attribute.
 *       However, the SafeStyle string *must be HTML-attribute-escaped* before
 *       interpolation.
 *   <li>Interpolated as the content of a {}-wrapped block within a stylesheet.
 *       '<' characters in the SafeStyle string *must be CSS-escaped* before
 *       interpolation. The SafeStyle string is also guaranteed not to be able
 *       to introduce new properties or elide existing ones.
 *   <li>Interpolated as the content of a {}-wrapped block within an HTML
 *       &lt;style&gt; element. '<' characters in the SafeStyle string
 *       *must be CSS-escaped* before interpolation.
 *   <li>Assigned to the style property of a DOM node. The SafeStyle string
 *       should not be escaped before being assigned to the property.
 * </ul>
 *
 * A SafeStyle may never contain literal angle brackets. Otherwise, it could
 * be unsafe to place a SafeStyle into a &lt;style&gt; tag (where it can't
 * be HTML escaped). For example, if the SafeStyle containing
 * "{@code font: 'foo &lt;style/&gt;&lt;script&gt;evil&lt;/script&gt;'}" were
 * interpolated within a &lt;style&gt; tag, this would then break out of the
 * style context into HTML.
 *
 * A SafeStyle may contain literal single or double quotes, and as such the
 * entire style string must be escaped when used in a style attribute (if
 * this were not the case, the string could contain a matching quote that
 * would escape from the style attribute).
 *
 * Values of this type must be composable, i.e. for any two values
 * `style1` and `style2` of this type,
 * {@code goog.html.SafeStyle.unwrap(style1) +
 * goog.html.SafeStyle.unwrap(style2)} must itself be a value that satisfies
 * the SafeStyle type constraint. This requirement implies that for any value
 * `style` of this type, `goog.html.SafeStyle.unwrap(style)` must
 * not end in a "property value" or "property name" context. For example,
 * a value of {@code background:url("} or {@code font-} would not satisfy the
 * SafeStyle contract. This is because concatenating such strings with a
 * second value that itself does not contain unsafe CSS can result in an
 * overall string that does. For example, if {@code javascript:evil())"} is
 * appended to {@code background:url("}, the resulting string may result in
 * the execution of a malicious script.
 *
 * TODO(mlourenco): Consider whether we should implement UTF-8 interchange
 * validity checks and blacklisting of newlines (including Unicode ones) and
 * other whitespace characters (\t, \f). Document here if so and also update
 * SafeStyle.fromConstant().
 *
 * The following example values comply with this type's contract:
 * <ul>
 *   <li><pre>width: 1em;</pre>
 *   <li><pre>height:1em;</pre>
 *   <li><pre>width: 1em;height: 1em;</pre>
 *   <li><pre>background:url('http://url');</pre>
 * </ul>
 * In addition, the empty string is safe for use in a CSS attribute.
 *
 * The following example values do NOT comply with this type's contract:
 * <ul>
 *   <li><pre>background: red</pre> (missing a trailing semi-colon)
 *   <li><pre>background:</pre> (missing a value and a trailing semi-colon)
 *   <li><pre>1em</pre> (missing an attribute name, which provides context for
 *       the value)
 * </ul>
 *
 * @see goog.html.SafeStyle#create
 * @see goog.html.SafeStyle#fromConstant
 * @see http://www.w3.org/TR/css3-syntax/
 * @constructor
 * @final
 * @struct
 * @implements {goog.string.TypedString}
 */
goog.html.SafeStyle = function() {
  /**
   * The contained value of this SafeStyle.  The field has a purposely
   * ugly name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @private {string}
   */
  this.privateDoNotAccessOrElseSafeStyleWrappedValue_ = '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.html.SafeStyle#unwrap
   * @const {!Object}
   * @private
   */
  this.SAFE_STYLE_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ =
      goog.html.SafeStyle.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_;
};


/**
 * @override
 * @const
 */
goog.html.SafeStyle.prototype.implementsGoogStringTypedString = true;


/**
 * Type marker for the SafeStyle type, used to implement additional
 * run-time type checking.
 * @const {!Object}
 * @private
 */
goog.html.SafeStyle.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ = {};


/**
 * Creates a SafeStyle object from a compile-time constant string.
 *
 * `style` should be in the format
 * {@code name: value; [name: value; ...]} and must not have any < or >
 * characters in it. This is so that SafeStyle's contract is preserved,
 * allowing the SafeStyle to correctly be interpreted as a sequence of CSS
 * declarations and without affecting the syntactic structure of any
 * surrounding CSS and HTML.
 *
 * This method performs basic sanity checks on the format of `style`
 * but does not constrain the format of `name` and `value`, except
 * for disallowing tag characters.
 *
 * @param {!goog.string.Const} style A compile-time-constant string from which
 *     to create a SafeStyle.
 * @return {!goog.html.SafeStyle} A SafeStyle object initialized to
 *     `style`.
 */
goog.html.SafeStyle.fromConstant = function(style) {
  var styleString = goog.string.Const.unwrap(style);
  if (styleString.length === 0) {
    return goog.html.SafeStyle.EMPTY;
  }
  goog.asserts.assert(
      goog.string.internal.endsWith(styleString, ';'),
      'Last character of style string is not \';\': ' + styleString);
  goog.asserts.assert(
      goog.string.internal.contains(styleString, ':'),
      'Style string must contain at least one \':\', to ' +
          'specify a "name: value" pair: ' + styleString);
  return goog.html.SafeStyle.createSafeStyleSecurityPrivateDoNotAccessOrElse(
      styleString);
};


/**
 * Returns this SafeStyle's value as a string.
 *
 * IMPORTANT: In code where it is security relevant that an object's type is
 * indeed `SafeStyle`, use `goog.html.SafeStyle.unwrap` instead of
 * this method. If in doubt, assume that it's security relevant. In particular,
 * note that goog.html functions which return a goog.html type do not guarantee
 * the returned instance is of the right type. For example:
 *
 * <pre>
 * var fakeSafeHtml = new String('fake');
 * fakeSafeHtml.__proto__ = goog.html.SafeHtml.prototype;
 * var newSafeHtml = goog.html.SafeHtml.htmlEscape(fakeSafeHtml);
 * // newSafeHtml is just an alias for fakeSafeHtml, it's passed through by
 * // goog.html.SafeHtml.htmlEscape() as fakeSafeHtml
 * // instanceof goog.html.SafeHtml.
 * </pre>
 *
 * @see goog.html.SafeStyle#unwrap
 * @override
 */
goog.html.SafeStyle.prototype.getTypedStringValue = function() {
  return this.privateDoNotAccessOrElseSafeStyleWrappedValue_;
};


if (goog.DEBUG) {
  /**
   * Returns a debug string-representation of this value.
   *
   * To obtain the actual string value wrapped in a SafeStyle, use
   * `goog.html.SafeStyle.unwrap`.
   *
   * @see goog.html.SafeStyle#unwrap
   * @override
   */
  goog.html.SafeStyle.prototype.toString = function() {
    return 'SafeStyle{' + this.privateDoNotAccessOrElseSafeStyleWrappedValue_ +
        '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed a
 * SafeStyle object, and returns its value.
 *
 * @param {!goog.html.SafeStyle} safeStyle The object to extract from.
 * @return {string} The safeStyle object's contained string, unless
 *     the run-time type check fails. In that case, `unwrap` returns an
 *     innocuous string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.html.SafeStyle.unwrap = function(safeStyle) {
  // Perform additional Run-time type-checking to ensure that
  // safeStyle is indeed an instance of the expected type.  This
  // provides some additional protection against security bugs due to
  // application code that disables type checks.
  // Specifically, the following checks are performed:
  // 1. The object is an instance of the expected type.
  // 2. The object is not an instance of a subclass.
  // 3. The object carries a type marker for the expected type. "Faking" an
  // object requires a reference to the type marker, which has names intended
  // to stand out in code reviews.
  if (safeStyle instanceof goog.html.SafeStyle &&
      safeStyle.constructor === goog.html.SafeStyle &&
      safeStyle.SAFE_STYLE_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ ===
          goog.html.SafeStyle.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_) {
    return safeStyle.privateDoNotAccessOrElseSafeStyleWrappedValue_;
  } else {
    goog.asserts.fail('expected object of type SafeStyle, got \'' +
        safeStyle + '\' of type ' + goog.typeOf(safeStyle));
    return 'type_error:SafeStyle';
  }
};


/**
 * Package-internal utility method to create SafeStyle instances.
 *
 * @param {string} style The string to initialize the SafeStyle object with.
 * @return {!goog.html.SafeStyle} The initialized SafeStyle object.
 * @package
 */
goog.html.SafeStyle.createSafeStyleSecurityPrivateDoNotAccessOrElse = function(
    style) {
  return new goog.html.SafeStyle().initSecurityPrivateDoNotAccessOrElse_(style);
};


/**
 * Called from createSafeStyleSecurityPrivateDoNotAccessOrElse(). This
 * method exists only so that the compiler can dead code eliminate static
 * fields (like EMPTY) when they're not accessed.
 * @param {string} style
 * @return {!goog.html.SafeStyle}
 * @private
 */
goog.html.SafeStyle.prototype.initSecurityPrivateDoNotAccessOrElse_ = function(
    style) {
  this.privateDoNotAccessOrElseSafeStyleWrappedValue_ = style;
  return this;
};


/**
 * A SafeStyle instance corresponding to the empty string.
 * @const {!goog.html.SafeStyle}
 */
goog.html.SafeStyle.EMPTY =
    goog.html.SafeStyle.createSafeStyleSecurityPrivateDoNotAccessOrElse('');


/**
 * The innocuous string generated by goog.html.SafeStyle.create when passed
 * an unsafe value.
 * @const {string}
 */
goog.html.SafeStyle.INNOCUOUS_STRING = 'zClosurez';


/**
 * A single property value.
 * @typedef {string|!goog.string.Const|!goog.html.SafeUrl}
 */
goog.html.SafeStyle.PropertyValue;


/**
 * Mapping of property names to their values.
 * We don't support numbers even though some values might be numbers (e.g.
 * line-height or 0 for any length). The reason is that most numeric values need
 * units (e.g. '1px') and allowing numbers could cause users forgetting about
 * them.
 * @typedef {!Object<string, ?goog.html.SafeStyle.PropertyValue|
 *     ?Array<!goog.html.SafeStyle.PropertyValue>>}
 */
goog.html.SafeStyle.PropertyMap;


/**
 * Creates a new SafeStyle object from the properties specified in the map.
 * @param {goog.html.SafeStyle.PropertyMap} map Mapping of property names to
 *     their values, for example {'margin': '1px'}. Names must consist of
 *     [-_a-zA-Z0-9]. Values might be strings consisting of
 *     [-,.'"%_!# a-zA-Z0-9[\]], where ", ', and [] must be properly balanced.
 *     We also allow simple functions like rgb() and url() which sanitizes its
 *     contents. Other values must be wrapped in goog.string.Const. URLs might
 *     be passed as goog.html.SafeUrl which will be wrapped into url(""). We
 *     also support array whose elements are joined with ' '. Null value causes
 *     skipping the property.
 * @return {!goog.html.SafeStyle}
 * @throws {Error} If invalid name is provided.
 * @throws {goog.asserts.AssertionError} If invalid value is provided. With
 *     disabled assertions, invalid value is replaced by
 *     goog.html.SafeStyle.INNOCUOUS_STRING.
 */
goog.html.SafeStyle.create = function(map) {
  var style = '';
  for (var name in map) {
    if (!/^[-_a-zA-Z0-9]+$/.test(name)) {
      throw new Error('Name allows only [-_a-zA-Z0-9], got: ' + name);
    }
    var value = map[name];
    if (value == null) {
      continue;
    }
    if (Array.isArray(value)) {
      value = goog.array.map(value, goog.html.SafeStyle.sanitizePropertyValue_)
                  .join(' ');
    } else {
      value = goog.html.SafeStyle.sanitizePropertyValue_(value);
    }
    style += name + ':' + value + ';';
  }
  if (!style) {
    return goog.html.SafeStyle.EMPTY;
  }
  return goog.html.SafeStyle.createSafeStyleSecurityPrivateDoNotAccessOrElse(
      style);
};


/**
 * Checks and converts value to string.
 * @param {!goog.html.SafeStyle.PropertyValue} value
 * @return {string}
 * @private
 */
goog.html.SafeStyle.sanitizePropertyValue_ = function(value) {
  if (value instanceof goog.html.SafeUrl) {
    var url = goog.html.SafeUrl.unwrap(value);
    return 'url("' + url.replace(/</g, '%3c').replace(/[\\"]/g, '\\$&') + '")';
  }
  var result = value instanceof goog.string.Const ?
      goog.string.Const.unwrap(value) :
      goog.html.SafeStyle.sanitizePropertyValueString_(String(value));
  // These characters can be used to change context and we don't want that even
  // with const values.
  if (/[{;}]/.test(result)) {
    throw new goog.asserts.AssertionError(
        'Value does not allow [{;}], got: %s.', [result]);
  }
  return result;
};


/**
 * Checks string value.
 * @param {string} value
 * @return {string}
 * @private
 */
goog.html.SafeStyle.sanitizePropertyValueString_ = function(value) {
  // Some CSS property values permit nested functions. We allow one level of
  // nesting, and all nested functions must also be in the FUNCTIONS_RE_ list.
  var valueWithoutFunctions =
      value.replace(goog.html.SafeStyle.FUNCTIONS_RE_, '$1')
          .replace(goog.html.SafeStyle.FUNCTIONS_RE_, '$1')
          .replace(goog.html.SafeStyle.URL_RE_, 'url');
  if (!goog.html.SafeStyle.VALUE_RE_.test(valueWithoutFunctions)) {
    goog.asserts.fail(
        'String value allows only ' + goog.html.SafeStyle.VALUE_ALLOWED_CHARS_ +
        ' and simple functions, got: ' + value);
    return goog.html.SafeStyle.INNOCUOUS_STRING;
  } else if (goog.html.SafeStyle.COMMENT_RE_.test(value)) {
    goog.asserts.fail('String value disallows comments, got: ' + value);
    return goog.html.SafeStyle.INNOCUOUS_STRING;
  } else if (!goog.html.SafeStyle.hasBalancedQuotes_(value)) {
    goog.asserts.fail('String value requires balanced quotes, got: ' + value);
    return goog.html.SafeStyle.INNOCUOUS_STRING;
  } else if (!goog.html.SafeStyle.hasBalancedSquareBrackets_(value)) {
    goog.asserts.fail(
        'String value requires balanced square brackets and one' +
        ' identifier per pair of brackets, got: ' + value);
    return goog.html.SafeStyle.INNOCUOUS_STRING;
  }
  return goog.html.SafeStyle.sanitizeUrl_(value);
};


/**
 * Checks that quotes (" and ') are properly balanced inside a string. Assumes
 * that neither escape (\) nor any other character that could result in
 * breaking out of a string parsing context are allowed;
 * see http://www.w3.org/TR/css3-syntax/#string-token-diagram.
 * @param {string} value Untrusted CSS property value.
 * @return {boolean} True if property value is safe with respect to quote
 *     balancedness.
 * @private
 */
goog.html.SafeStyle.hasBalancedQuotes_ = function(value) {
  var outsideSingle = true;
  var outsideDouble = true;
  for (var i = 0; i < value.length; i++) {
    var c = value.charAt(i);
    if (c == "'" && outsideDouble) {
      outsideSingle = !outsideSingle;
    } else if (c == '"' && outsideSingle) {
      outsideDouble = !outsideDouble;
    }
  }
  return outsideSingle && outsideDouble;
};


/**
 * Checks that square brackets ([ and ]) are properly balanced inside a string,
 * and that the content in the square brackets is one ident-token;
 * see https://www.w3.org/TR/css-syntax-3/#ident-token-diagram.
 * For practicality, and in line with other restrictions posed on SafeStyle
 * strings, we restrict the character set allowable in the ident-token to
 * [-_a-zA-Z0-9].
 * @param {string} value Untrusted CSS property value.
 * @return {boolean} True if property value is safe with respect to square
 *     bracket balancedness.
 * @private
 */
goog.html.SafeStyle.hasBalancedSquareBrackets_ = function(value) {
  var outside = true;
  var tokenRe = /^[-_a-zA-Z0-9]$/;
  for (var i = 0; i < value.length; i++) {
    var c = value.charAt(i);
    if (c == ']') {
      if (outside) return false;  // Unbalanced ].
      outside = true;
    } else if (c == '[') {
      if (!outside) return false;  // No nesting.
      outside = false;
    } else if (!outside && !tokenRe.test(c)) {
      return false;
    }
  }
  return outside;
};


/**
 * Characters allowed in goog.html.SafeStyle.VALUE_RE_.
 * @private {string}
 */
goog.html.SafeStyle.VALUE_ALLOWED_CHARS_ = '[-,."\'%_!# a-zA-Z0-9\\[\\]]';


/**
 * Regular expression for safe values.
 *
 * Quotes (" and ') are allowed, but a check must be done elsewhere to ensure
 * they're balanced.
 *
 * Square brackets ([ and ]) are allowed, but a check must be done elsewhere
 * to ensure they're balanced. The content inside a pair of square brackets must
 * be one alphanumeric identifier.
 *
 * ',' allows multiple values to be assigned to the same property
 * (e.g. background-attachment or font-family) and hence could allow
 * multiple values to get injected, but that should pose no risk of XSS.
 *
 * The expression checks only for XSS safety, not for CSS validity.
 * @const {!RegExp}
 * @private
 */
goog.html.SafeStyle.VALUE_RE_ =
    new RegExp('^' + goog.html.SafeStyle.VALUE_ALLOWED_CHARS_ + '+$');


/**
 * Regular expression for url(). We support URLs allowed by
 * https://www.w3.org/TR/css-syntax-3/#url-token-diagram without using escape
 * sequences. Use percent-encoding if you need to use special characters like
 * backslash.
 * @private @const {!RegExp}
 */
goog.html.SafeStyle.URL_RE_ = new RegExp(
    '\\b(url\\([ \t\n]*)(' +
        '\'[ -&(-\\[\\]-~]*\'' +  // Printable characters except ' and \.
        '|"[ !#-\\[\\]-~]*"' +    // Printable characters except " and \.
        '|[!#-&*-\\[\\]-~]*' +    // Printable characters except [ "'()\\].
        ')([ \t\n]*\\))',
    'g');

/**
 * Names of functions allowed in FUNCTIONS_RE_.
 * @private @const {!Array<string>}
 */
goog.html.SafeStyle.ALLOWED_FUNCTIONS_ = [
  'calc',
  'cubic-bezier',
  'fit-content',
  'hsl',
  'hsla',
  'linear-gradient',
  'matrix',
  'minmax',
  'repeat',
  'rgb',
  'rgba',
  '(rotate|scale|translate)(X|Y|Z|3d)?',
];


/**
 * Regular expression for simple functions.
 * @private @const {!RegExp}
 */
goog.html.SafeStyle.FUNCTIONS_RE_ = new RegExp(
    '\\b(' + goog.html.SafeStyle.ALLOWED_FUNCTIONS_.join('|') + ')' +
        '\\([-+*/0-9a-z.%\\[\\], ]+\\)',
    'g');


/**
 * Regular expression for comments. These are disallowed in CSS property values.
 * @private @const {!RegExp}
 */
goog.html.SafeStyle.COMMENT_RE_ = /\/\*/;


/**
 * Sanitize URLs inside url().
 *
 * NOTE: We could also consider using CSS.escape once that's available in the
 * browsers. However, loosely matching URL e.g. with url\(.*\) and then escaping
 * the contents would result in a slightly different language than CSS leading
 * to confusion of users. E.g. url(")") is valid in CSS but it would be invalid
 * as seen by our parser. On the other hand, url(\) is invalid in CSS but our
 * parser would be fine with it.
 *
 * @param {string} value Untrusted CSS property value.
 * @return {string}
 * @private
 */
goog.html.SafeStyle.sanitizeUrl_ = function(value) {
  return value.replace(
      goog.html.SafeStyle.URL_RE_, function(match, before, url, after) {
        var quote = '';
        url = url.replace(/^(['"])(.*)\1$/, function(match, start, inside) {
          quote = start;
          return inside;
        });
        var sanitized = goog.html.SafeUrl.sanitize(url).getTypedStringValue();
        return before + quote + sanitized + quote + after;
      });
};


/**
 * Creates a new SafeStyle object by concatenating the values.
 * @param {...(!goog.html.SafeStyle|!Array<!goog.html.SafeStyle>)} var_args
 *     SafeStyles to concatenate.
 * @return {!goog.html.SafeStyle}
 */
goog.html.SafeStyle.concat = function(var_args) {
  var style = '';

  /**
   * @param {!goog.html.SafeStyle|!Array<!goog.html.SafeStyle>} argument
   */
  var addArgument = function(argument) {
    if (Array.isArray(argument)) {
      goog.array.forEach(argument, addArgument);
    } else {
      style += goog.html.SafeStyle.unwrap(argument);
    }
  };

  goog.array.forEach(arguments, addArgument);
  if (!style) {
    return goog.html.SafeStyle.EMPTY;
  }
  return goog.html.SafeStyle.createSafeStyleSecurityPrivateDoNotAccessOrElse(
      style);
};

//third_party/javascript/closure/html/safestylesheet.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview The SafeStyleSheet type and its builders.
 *
 * TODO(xtof): Link to document stating type contract.
 */

goog.provide('goog.html.SafeStyleSheet');

goog.require('goog.array');
goog.require('goog.asserts');
goog.require('goog.html.SafeStyle');
goog.require('goog.object');
goog.require('goog.string.Const');
goog.require('goog.string.TypedString');
goog.require('goog.string.internal');



/**
 * A string-like object which represents a CSS style sheet and that carries the
 * security type contract that its value, as a string, will not cause untrusted
 * script execution (XSS) when evaluated as CSS in a browser.
 *
 * Instances of this type must be created via the factory method
 * `goog.html.SafeStyleSheet.fromConstant` and not by invoking its
 * constructor. The constructor intentionally takes no parameters and the type
 * is immutable; hence only a default instance corresponding to the empty string
 * can be obtained via constructor invocation.
 *
 * A SafeStyleSheet's string representation can safely be interpolated as the
 * content of a style element within HTML. The SafeStyleSheet string should
 * not be escaped before interpolation.
 *
 * Values of this type must be composable, i.e. for any two values
 * `styleSheet1` and `styleSheet2` of this type,
 * {@code goog.html.SafeStyleSheet.unwrap(styleSheet1) +
 * goog.html.SafeStyleSheet.unwrap(styleSheet2)} must itself be a value that
 * satisfies the SafeStyleSheet type constraint. This requirement implies that
 * for any value `styleSheet` of this type,
 * `goog.html.SafeStyleSheet.unwrap(styleSheet1)` must end in
 * "beginning of rule" context.

 * A SafeStyleSheet can be constructed via security-reviewed unchecked
 * conversions. In this case producers of SafeStyleSheet must ensure themselves
 * that the SafeStyleSheet does not contain unsafe script. Note in particular
 * that {@code &lt;} is dangerous, even when inside CSS strings, and so should
 * always be forbidden or CSS-escaped in user controlled input. For example, if
 * {@code &lt;/style&gt;&lt;script&gt;evil&lt;/script&gt;"} were interpolated
 * inside a CSS string, it would break out of the context of the original
 * style element and `evil` would execute. Also note that within an HTML
 * style (raw text) element, HTML character references, such as
 * {@code &amp;lt;}, are not allowed. See
 *
 http://www.w3.org/TR/html5/scripting-1.html#restrictions-for-contents-of-script-elements
 * (similar considerations apply to the style element).
 *
 * @see goog.html.SafeStyleSheet#fromConstant
 * @constructor
 * @final
 * @struct
 * @implements {goog.string.TypedString}
 */
goog.html.SafeStyleSheet = function() {
  /**
   * The contained value of this SafeStyleSheet.  The field has a purposely
   * ugly name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @private {string}
   */
  this.privateDoNotAccessOrElseSafeStyleSheetWrappedValue_ = '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.html.SafeStyleSheet#unwrap
   * @const {!Object}
   * @private
   */
  this.SAFE_STYLE_SHEET_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ =
      goog.html.SafeStyleSheet.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_;
};


/**
 * @override
 * @const
 */
goog.html.SafeStyleSheet.prototype.implementsGoogStringTypedString = true;


/**
 * Type marker for the SafeStyleSheet type, used to implement additional
 * run-time type checking.
 * @const {!Object}
 * @private
 */
goog.html.SafeStyleSheet.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ = {};


/**
 * Creates a style sheet consisting of one selector and one style definition.
 * Use {@link goog.html.SafeStyleSheet.concat} to create longer style sheets.
 * This function doesn't support @import, @media and similar constructs.
 * @param {string} selector CSS selector, e.g. '#id' or 'tag .class, #id'. We
 *     support CSS3 selectors: https://w3.org/TR/css3-selectors/#selectors.
 * @param {!goog.html.SafeStyle.PropertyMap|!goog.html.SafeStyle} style Style
 *     definition associated with the selector.
 * @return {!goog.html.SafeStyleSheet}
 * @throws {Error} If invalid selector is provided.
 */
goog.html.SafeStyleSheet.createRule = function(selector, style) {
  if (goog.string.internal.contains(selector, '<')) {
    throw new Error('Selector does not allow \'<\', got: ' + selector);
  }

  // Remove strings.
  var selectorToCheck =
      selector.replace(/('|")((?!\1)[^\r\n\f\\]|\\[\s\S])*\1/g, '');

  // Check characters allowed in CSS3 selectors.
  if (!/^[-_a-zA-Z0-9#.:* ,>+~[\]()=^$|]+$/.test(selectorToCheck)) {
    throw new Error(
        'Selector allows only [-_a-zA-Z0-9#.:* ,>+~[\\]()=^$|] and ' +
        'strings, got: ' + selector);
  }

  // Check balanced () and [].
  if (!goog.html.SafeStyleSheet.hasBalancedBrackets_(selectorToCheck)) {
    throw new Error('() and [] in selector must be balanced, got: ' + selector);
  }

  if (!(style instanceof goog.html.SafeStyle)) {
    style = goog.html.SafeStyle.create(style);
  }
  var styleSheet = selector + '{' +
      goog.html.SafeStyle.unwrap(style).replace(/</g, '\\3C ') + '}';
  return goog.html.SafeStyleSheet
      .createSafeStyleSheetSecurityPrivateDoNotAccessOrElse(styleSheet);
};


/**
 * Checks if a string has balanced () and [] brackets.
 * @param {string} s String to check.
 * @return {boolean}
 * @private
 */
goog.html.SafeStyleSheet.hasBalancedBrackets_ = function(s) {
  var brackets = {'(': ')', '[': ']'};
  var expectedBrackets = [];
  for (var i = 0; i < s.length; i++) {
    var ch = s[i];
    if (brackets[ch]) {
      expectedBrackets.push(brackets[ch]);
    } else if (goog.object.contains(brackets, ch)) {
      if (expectedBrackets.pop() != ch) {
        return false;
      }
    }
  }
  return expectedBrackets.length == 0;
};


/**
 * Creates a new SafeStyleSheet object by concatenating values.
 * @param {...(!goog.html.SafeStyleSheet|!Array<!goog.html.SafeStyleSheet>)}
 *     var_args Values to concatenate.
 * @return {!goog.html.SafeStyleSheet}
 */
goog.html.SafeStyleSheet.concat = function(var_args) {
  var result = '';

  /**
   * @param {!goog.html.SafeStyleSheet|!Array<!goog.html.SafeStyleSheet>}
   *     argument
   */
  var addArgument = function(argument) {
    if (Array.isArray(argument)) {
      goog.array.forEach(argument, addArgument);
    } else {
      result += goog.html.SafeStyleSheet.unwrap(argument);
    }
  };

  goog.array.forEach(arguments, addArgument);
  return goog.html.SafeStyleSheet
      .createSafeStyleSheetSecurityPrivateDoNotAccessOrElse(result);
};


/**
 * Creates a SafeStyleSheet object from a compile-time constant string.
 *
 * `styleSheet` must not have any &lt; characters in it, so that
 * the syntactic structure of the surrounding HTML is not affected.
 *
 * @param {!goog.string.Const} styleSheet A compile-time-constant string from
 *     which to create a SafeStyleSheet.
 * @return {!goog.html.SafeStyleSheet} A SafeStyleSheet object initialized to
 *     `styleSheet`.
 */
goog.html.SafeStyleSheet.fromConstant = function(styleSheet) {
  var styleSheetString = goog.string.Const.unwrap(styleSheet);
  if (styleSheetString.length === 0) {
    return goog.html.SafeStyleSheet.EMPTY;
  }
  // > is a valid character in CSS selectors and there's no strict need to
  // block it if we already block <.
  goog.asserts.assert(
      !goog.string.internal.contains(styleSheetString, '<'),
      'Forbidden \'<\' character in style sheet string: ' + styleSheetString);
  return goog.html.SafeStyleSheet
      .createSafeStyleSheetSecurityPrivateDoNotAccessOrElse(styleSheetString);
};


/**
 * Returns this SafeStyleSheet's value as a string.
 *
 * IMPORTANT: In code where it is security relevant that an object's type is
 * indeed `SafeStyleSheet`, use `goog.html.SafeStyleSheet.unwrap`
 * instead of this method. If in doubt, assume that it's security relevant. In
 * particular, note that goog.html functions which return a goog.html type do
 * not guarantee the returned instance is of the right type. For example:
 *
 * <pre>
 * var fakeSafeHtml = new String('fake');
 * fakeSafeHtml.__proto__ = goog.html.SafeHtml.prototype;
 * var newSafeHtml = goog.html.SafeHtml.htmlEscape(fakeSafeHtml);
 * // newSafeHtml is just an alias for fakeSafeHtml, it's passed through by
 * // goog.html.SafeHtml.htmlEscape() as fakeSafeHtml
 * // instanceof goog.html.SafeHtml.
 * </pre>
 *
 * @see goog.html.SafeStyleSheet#unwrap
 * @override
 */
goog.html.SafeStyleSheet.prototype.getTypedStringValue = function() {
  return this.privateDoNotAccessOrElseSafeStyleSheetWrappedValue_;
};


if (goog.DEBUG) {
  /**
   * Returns a debug string-representation of this value.
   *
   * To obtain the actual string value wrapped in a SafeStyleSheet, use
   * `goog.html.SafeStyleSheet.unwrap`.
   *
   * @see goog.html.SafeStyleSheet#unwrap
   * @override
   */
  goog.html.SafeStyleSheet.prototype.toString = function() {
    return 'SafeStyleSheet{' +
        this.privateDoNotAccessOrElseSafeStyleSheetWrappedValue_ + '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed a
 * SafeStyleSheet object, and returns its value.
 *
 * @param {!goog.html.SafeStyleSheet} safeStyleSheet The object to extract from.
 * @return {string} The safeStyleSheet object's contained string, unless
 *     the run-time type check fails. In that case, `unwrap` returns an
 *     innocuous string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.html.SafeStyleSheet.unwrap = function(safeStyleSheet) {
  // Perform additional Run-time type-checking to ensure that
  // safeStyleSheet is indeed an instance of the expected type.  This
  // provides some additional protection against security bugs due to
  // application code that disables type checks.
  // Specifically, the following checks are performed:
  // 1. The object is an instance of the expected type.
  // 2. The object is not an instance of a subclass.
  // 3. The object carries a type marker for the expected type. "Faking" an
  // object requires a reference to the type marker, which has names intended
  // to stand out in code reviews.
  if (safeStyleSheet instanceof goog.html.SafeStyleSheet &&
      safeStyleSheet.constructor === goog.html.SafeStyleSheet &&
      safeStyleSheet
              .SAFE_STYLE_SHEET_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ ===
          goog.html.SafeStyleSheet.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_) {
    return safeStyleSheet.privateDoNotAccessOrElseSafeStyleSheetWrappedValue_;
  } else {
    goog.asserts.fail('expected object of type SafeStyleSheet, got \'' +
        safeStyleSheet + '\' of type ' + goog.typeOf(safeStyleSheet));
    return 'type_error:SafeStyleSheet';
  }
};


/**
 * Package-internal utility method to create SafeStyleSheet instances.
 *
 * @param {string} styleSheet The string to initialize the SafeStyleSheet
 *     object with.
 * @return {!goog.html.SafeStyleSheet} The initialized SafeStyleSheet object.
 * @package
 */
goog.html.SafeStyleSheet.createSafeStyleSheetSecurityPrivateDoNotAccessOrElse =
    function(styleSheet) {
  return new goog.html.SafeStyleSheet().initSecurityPrivateDoNotAccessOrElse_(
      styleSheet);
};


/**
 * Called from createSafeStyleSheetSecurityPrivateDoNotAccessOrElse(). This
 * method exists only so that the compiler can dead code eliminate static
 * fields (like EMPTY) when they're not accessed.
 * @param {string} styleSheet
 * @return {!goog.html.SafeStyleSheet}
 * @private
 */
goog.html.SafeStyleSheet.prototype.initSecurityPrivateDoNotAccessOrElse_ =
    function(styleSheet) {
  this.privateDoNotAccessOrElseSafeStyleSheetWrappedValue_ = styleSheet;
  return this;
};


/**
 * A SafeStyleSheet instance corresponding to the empty string.
 * @const {!goog.html.SafeStyleSheet}
 */
goog.html.SafeStyleSheet.EMPTY =
    goog.html.SafeStyleSheet
        .createSafeStyleSheetSecurityPrivateDoNotAccessOrElse('');

//third_party/javascript/closure/labs/useragent/util.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Utilities used by goog.labs.userAgent tools. These functions
 * should not be used outside of goog.labs.userAgent.*.
 *
 * @visibility {//javascript/abc/libs/objects3d:__subpackages__}
 * @visibility {//javascript/closure/dom:__subpackages__}
 * @visibility {//javascript/closure/style:__pkg__}
 * @visibility {//javascript/closure/testing:__pkg__}
 * @visibility {//javascript/closure/useragent:__subpackages__}
 * @visibility {//testing/puppet/modules:__pkg__}
 * @visibility {:util_legacy_users}
 */

goog.provide('goog.labs.userAgent.util');

goog.require('goog.string.internal');


/**
 * Gets the native userAgent string from navigator if it exists.
 * If navigator or navigator.userAgent string is missing, returns an empty
 * string.
 * @return {string}
 * @private
 */
goog.labs.userAgent.util.getNativeUserAgentString_ = function() {
  var navigator = goog.labs.userAgent.util.getNavigator_();
  if (navigator) {
    var userAgent = navigator.userAgent;
    if (userAgent) {
      return userAgent;
    }
  }
  return '';
};


/**
 * Getter for the native navigator.
 * This is a separate function so it can be stubbed out in testing.
 * @return {!Navigator}
 * @private
 */
goog.labs.userAgent.util.getNavigator_ = function() {
  return goog.global.navigator;
};


/**
 * A possible override for applications which wish to not check
 * navigator.userAgent but use a specified value for detection instead.
 * @private {string}
 */
goog.labs.userAgent.util.userAgent_ =
    goog.labs.userAgent.util.getNativeUserAgentString_();


/**
 * Applications may override browser detection on the built in
 * navigator.userAgent object by setting this string. Set to null to use the
 * browser object instead.
 * @param {?string=} opt_userAgent The User-Agent override.
 */
goog.labs.userAgent.util.setUserAgent = function(opt_userAgent) {
  goog.labs.userAgent.util.userAgent_ =
      opt_userAgent || goog.labs.userAgent.util.getNativeUserAgentString_();
};


/**
 * @return {string} The user agent string.
 */
goog.labs.userAgent.util.getUserAgent = function() {
  return goog.labs.userAgent.util.userAgent_;
};


/**
 * @param {string} str
 * @return {boolean} Whether the user agent contains the given string.
 */
goog.labs.userAgent.util.matchUserAgent = function(str) {
  var userAgent = goog.labs.userAgent.util.getUserAgent();
  return goog.string.internal.contains(userAgent, str);
};


/**
 * @param {string} str
 * @return {boolean} Whether the user agent contains the given string, ignoring
 *     case.
 */
goog.labs.userAgent.util.matchUserAgentIgnoreCase = function(str) {
  var userAgent = goog.labs.userAgent.util.getUserAgent();
  return goog.string.internal.caseInsensitiveContains(userAgent, str);
};


/**
 * Parses the user agent into tuples for each section.
 * @param {string} userAgent
 * @return {!Array<!Array<string>>} Tuples of key, version, and the contents
 *     of the parenthetical.
 */
goog.labs.userAgent.util.extractVersionTuples = function(userAgent) {
  // Matches each section of a user agent string.
  // Example UA:
  // Mozilla/5.0 (iPad; U; CPU OS 3_2_1 like Mac OS X; en-us)
  // AppleWebKit/531.21.10 (KHTML, like Gecko) Mobile/7B405
  // This has three version tuples: Mozilla, AppleWebKit, and Mobile.

  var versionRegExp = new RegExp(
      // Key. Note that a key may have a space.
      // (i.e. 'Mobile Safari' in 'Mobile Safari/5.0')
      '(\\w[\\w ]+)' +

          '/' +                // slash
          '([^\\s]+)' +        // version (i.e. '5.0b')
          '\\s*' +             // whitespace
          '(?:\\((.*?)\\))?',  // parenthetical info. parentheses not matched.
      'g');

  var data = [];
  var match;

  // Iterate and collect the version tuples.  Each iteration will be the
  // next regex match.
  while (match = versionRegExp.exec(userAgent)) {
    data.push([
      match[1],  // key
      match[2],  // value
      // || undefined as this is not undefined in IE7 and IE8
      match[3] || undefined  // info
    ]);
  }

  return data;
};

//third_party/javascript/closure/labs/useragent/browser.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */

/**
 * @fileoverview Closure user agent detection (Browser).
 * @see <a href="http://www.useragentstring.com/">User agent strings</a>
 * For more information on rendering engine, platform, or device see the other
 * sub-namespaces in goog.labs.userAgent, goog.labs.userAgent.platform,
 * goog.labs.userAgent.device respectively.)
 */

goog.provide('goog.labs.userAgent.browser');

goog.require('goog.array');
goog.require('goog.labs.userAgent.util');
goog.require('goog.object');
goog.require('goog.string.internal');


// TODO(nnaze): Refactor to remove excessive exclusion logic in matching
// functions.


/**
 * @return {boolean} Whether the user's browser is Opera.  Note: Chromium
 *     based Opera (Opera 15+) is detected as Chrome to avoid unnecessary
 *     special casing.
 * @private
 */
goog.labs.userAgent.browser.matchOpera_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Opera');
};


/**
 * @return {boolean} Whether the user's browser is IE.
 * @private
 */
goog.labs.userAgent.browser.matchIE_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Trident') ||
      goog.labs.userAgent.util.matchUserAgent('MSIE');
};


/**
 * @return {boolean} Whether the user's browser is Edge. This refers to EdgeHTML
 * based Edge.
 * @private
 */
goog.labs.userAgent.browser.matchEdgeHtml_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Edge');
};


/**
 * @return {boolean} Whether the user's browser is Chromium based Edge.
 * @private
 */
goog.labs.userAgent.browser.matchEdgeChromium_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Edg/');
};


/**
 * @return {boolean} Whether the user's browser is Chromium based Opera.
 * @private
 */
goog.labs.userAgent.browser.matchOperaChromium_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('OPR');
};


/**
 * @return {boolean} Whether the user's browser is Firefox.
 * @private
 */
goog.labs.userAgent.browser.matchFirefox_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Firefox') ||
      goog.labs.userAgent.util.matchUserAgent('FxiOS');
};


/**
 * @return {boolean} Whether the user's browser is Safari.
 * @private
 */
goog.labs.userAgent.browser.matchSafari_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Safari') &&
      !(goog.labs.userAgent.browser.matchChrome_() ||
        goog.labs.userAgent.browser.matchCoast_() ||
        goog.labs.userAgent.browser.matchOpera_() ||
        goog.labs.userAgent.browser.matchEdgeHtml_() ||
        goog.labs.userAgent.browser.matchEdgeChromium_() ||
        goog.labs.userAgent.browser.matchOperaChromium_() ||
        goog.labs.userAgent.browser.matchFirefox_() ||
        goog.labs.userAgent.browser.isSilk() ||
        goog.labs.userAgent.util.matchUserAgent('Android'));
};


/**
 * @return {boolean} Whether the user's browser is Coast (Opera's Webkit-based
 *     iOS browser).
 * @private
 */
goog.labs.userAgent.browser.matchCoast_ = function() {
  return goog.labs.userAgent.util.matchUserAgent('Coast');
};


/**
 * @return {boolean} Whether the user's browser is iOS Webview.
 * @private
 */
goog.labs.userAgent.browser.matchIosWebview_ = function() {
  // iOS Webview does not show up as Chrome or Safari. Also check for Opera's
  // WebKit-based iOS browser, Coast.
  return (goog.labs.userAgent.util.matchUserAgent('iPad') ||
          goog.labs.userAgent.util.matchUserAgent('iPhone')) &&
      !goog.labs.userAgent.browser.matchSafari_() &&
      !goog.labs.userAgent.browser.matchChrome_() &&
      !goog.labs.userAgent.browser.matchCoast_() &&
      !goog.labs.userAgent.browser.matchFirefox_() &&
      goog.labs.userAgent.util.matchUserAgent('AppleWebKit');
};


/**
 * @return {boolean} Whether the user's browser is any Chromium browser. This
 * returns true for Chrome, Opera 15+, and Edge Chromium.
 * @private
 */
goog.labs.userAgent.browser.matchChrome_ = function() {
  return (goog.labs.userAgent.util.matchUserAgent('Chrome') ||
          goog.labs.userAgent.util.matchUserAgent('CriOS')) &&
      !goog.labs.userAgent.browser.matchEdgeHtml_();
};


/**
 * @return {boolean} Whether the user's browser is the Android browser.
 * @private
 */
goog.labs.userAgent.browser.matchAndroidBrowser_ = function() {
  // Android can appear in the user agent string for Chrome on Android.
  // This is not the Android standalone browser if it does.
  return goog.labs.userAgent.util.matchUserAgent('Android') &&
      !(goog.labs.userAgent.browser.isChrome() ||
        goog.labs.userAgent.browser.isFirefox() ||
        goog.labs.userAgent.browser.isOpera() ||
        goog.labs.userAgent.browser.isSilk());
};


/**
 * @return {boolean} Whether the user's browser is Opera.
 */
goog.labs.userAgent.browser.isOpera = goog.labs.userAgent.browser.matchOpera_;


/**
 * @return {boolean} Whether the user's browser is IE.
 */
goog.labs.userAgent.browser.isIE = goog.labs.userAgent.browser.matchIE_;


/**
 * @return {boolean} Whether the user's browser is EdgeHTML based Edge.
 */
goog.labs.userAgent.browser.isEdge = goog.labs.userAgent.browser.matchEdgeHtml_;


/**
 * @return {boolean} Whether the user's browser is Chromium based Edge.
 */
goog.labs.userAgent.browser.isEdgeChromium =
    goog.labs.userAgent.browser.matchEdgeChromium_;

/**
 * @return {boolean} Whether the user's browser is Chromium based Opera.
 */
goog.labs.userAgent.browser.isOperaChromium =
    goog.labs.userAgent.browser.matchOperaChromium_;

/**
 * @return {boolean} Whether the user's browser is Firefox.
 */
goog.labs.userAgent.browser.isFirefox =
    goog.labs.userAgent.browser.matchFirefox_;


/**
 * @return {boolean} Whether the user's browser is Safari.
 */
goog.labs.userAgent.browser.isSafari = goog.labs.userAgent.browser.matchSafari_;


/**
 * @return {boolean} Whether the user's browser is Coast (Opera's Webkit-based
 *     iOS browser).
 */
goog.labs.userAgent.browser.isCoast = goog.labs.userAgent.browser.matchCoast_;


/**
 * @return {boolean} Whether the user's browser is iOS Webview.
 */
goog.labs.userAgent.browser.isIosWebview =
    goog.labs.userAgent.browser.matchIosWebview_;


/**
 * @return {boolean} Whether the user's browser is any Chromium based browser (
 * Chrome, Blink-based Opera (15+) and Edge Chromium).
 */
goog.labs.userAgent.browser.isChrome = goog.labs.userAgent.browser.matchChrome_;


/**
 * @return {boolean} Whether the user's browser is the Android browser.
 */
goog.labs.userAgent.browser.isAndroidBrowser =
    goog.labs.userAgent.browser.matchAndroidBrowser_;


/**
 * For more information, see:
 * http://docs.aws.amazon.com/silk/latest/developerguide/user-agent.html
 * @return {boolean} Whether the user's browser is Silk.
 */
goog.labs.userAgent.browser.isSilk = function() {
  return goog.labs.userAgent.util.matchUserAgent('Silk');
};


/**
 * @return {string} The browser version or empty string if version cannot be
 *     determined. Note that for Internet Explorer, this returns the version of
 *     the browser, not the version of the rendering engine. (IE 8 in
 *     compatibility mode will return 8.0 rather than 7.0. To determine the
 *     rendering engine version, look at document.documentMode instead. See
 *     http://msdn.microsoft.com/en-us/library/cc196988(v=vs.85).aspx for more
 *     details.)
 */
goog.labs.userAgent.browser.getVersion = function() {
  var userAgentString = goog.labs.userAgent.util.getUserAgent();
  // Special case IE since IE's version is inside the parenthesis and
  // without the '/'.
  if (goog.labs.userAgent.browser.isIE()) {
    return goog.labs.userAgent.browser.getIEVersion_(userAgentString);
  }

  var versionTuples =
      goog.labs.userAgent.util.extractVersionTuples(userAgentString);

  // Construct a map for easy lookup.
  var versionMap = {};
  goog.array.forEach(versionTuples, function(tuple) {
    // Note that the tuple is of length three, but we only care about the
    // first two.
    var key = tuple[0];
    var value = tuple[1];
    versionMap[key] = value;
  });

  var versionMapHasKey = goog.partial(goog.object.containsKey, versionMap);

  // Gives the value with the first key it finds, otherwise empty string.
  function lookUpValueWithKeys(keys) {
    var key = goog.array.find(keys, versionMapHasKey);
    return versionMap[key] || '';
  }

  // Check Opera before Chrome since Opera 15+ has "Chrome" in the string.
  // See
  // http://my.opera.com/ODIN/blog/2013/07/15/opera-user-agent-strings-opera-15-and-beyond
  if (goog.labs.userAgent.browser.isOpera()) {
    // Opera 10 has Version/10.0 but Opera/9.8, so look for "Version" first.
    // Opera uses 'OPR' for more recent UAs.
    return lookUpValueWithKeys(['Version', 'Opera']);
  }

  // Check Edge before Chrome since it has Chrome in the string.
  if (goog.labs.userAgent.browser.isEdge()) {
    return lookUpValueWithKeys(['Edge']);
  }

  // Check Chromium Edge before Chrome since it has Chrome in the string.
  if (goog.labs.userAgent.browser.isEdgeChromium()) {
    return lookUpValueWithKeys(['Edg']);
  }

  if (goog.labs.userAgent.browser.isChrome()) {
    return lookUpValueWithKeys(['Chrome', 'CriOS', 'HeadlessChrome']);
  }

  // Usually products browser versions are in the third tuple after "Mozilla"
  // and the engine.
  var tuple = versionTuples[2];
  return tuple && tuple[1] || '';
};


/**
 * @param {string|number} version The version to check.
 * @return {boolean} Whether the browser version is higher or the same as the
 *     given version.
 */
goog.labs.userAgent.browser.isVersionOrHigher = function(version) {
  return goog.string.internal.compareVersions(
             goog.labs.userAgent.browser.getVersion(), version) >= 0;
};


/**
 * Determines IE version. More information:
 * http://msdn.microsoft.com/en-us/library/ie/bg182625(v=vs.85).aspx#uaString
 * http://msdn.microsoft.com/en-us/library/hh869301(v=vs.85).aspx
 * http://blogs.msdn.com/b/ie/archive/2010/03/23/introducing-ie9-s-user-agent-string.aspx
 * http://blogs.msdn.com/b/ie/archive/2009/01/09/the-internet-explorer-8-user-agent-string-updated-edition.aspx
 *
 * @param {string} userAgent the User-Agent.
 * @return {string}
 * @private
 */
goog.labs.userAgent.browser.getIEVersion_ = function(userAgent) {
  // IE11 may identify itself as MSIE 9.0 or MSIE 10.0 due to an IE 11 upgrade
  // bug. Example UA:
  // Mozilla/5.0 (MSIE 9.0; Windows NT 6.1; WOW64; Trident/7.0; rv:11.0)
  // like Gecko.
  // See http://www.whatismybrowser.com/developers/unknown-user-agent-fragments.
  var rv = /rv: *([\d\.]*)/.exec(userAgent);
  if (rv && rv[1]) {
    return rv[1];
  }

  var version = '';
  var msie = /MSIE +([\d\.]+)/.exec(userAgent);
  if (msie && msie[1]) {
    // IE in compatibility mode usually identifies itself as MSIE 7.0; in this
    // case, use the Trident version to determine the version of IE. For more
    // details, see the links above.
    var tridentVersion = /Trident\/(\d.\d)/.exec(userAgent);
    if (msie[1] == '7.0') {
      if (tridentVersion && tridentVersion[1]) {
        switch (tridentVersion[1]) {
          case '4.0':
            version = '8.0';
            break;
          case '5.0':
            version = '9.0';
            break;
          case '6.0':
            version = '10.0';
            break;
          case '7.0':
            version = '11.0';
            break;
        }
      } else {
        version = '7.0';
      }
    } else {
      version = msie[1];
    }
  }
  return version;
};

//third_party/javascript/closure/html/safehtml.js
/**
 * @license
 * Copyright The Closure Library Authors.
 * SPDX-License-Identifier: Apache-2.0
 */


/**
 * @fileoverview The SafeHtml type and its builders.
 *
 * TODO(xtof): Link to document stating type contract.
 */

goog.provide('goog.html.SafeHtml');

goog.require('goog.array');
goog.require('goog.asserts');
goog.require('goog.dom.TagName');
goog.require('goog.dom.tags');
goog.require('goog.html.SafeScript');
goog.require('goog.html.SafeStyle');
goog.require('goog.html.SafeStyleSheet');
goog.require('goog.html.SafeUrl');
goog.require('goog.html.TrustedResourceUrl');
goog.require('goog.html.trustedtypes');
goog.require('goog.i18n.bidi.Dir');
goog.require('goog.i18n.bidi.DirectionalString');
goog.require('goog.labs.userAgent.browser');
goog.require('goog.object');
goog.require('goog.string.Const');
goog.require('goog.string.TypedString');
goog.require('goog.string.internal');



/**
 * A string that is safe to use in HTML context in DOM APIs and HTML documents.
 *
 * A SafeHtml is a string-like object that carries the security type contract
 * that its value as a string will not cause untrusted script execution when
 * evaluated as HTML in a browser.
 *
 * Values of this type are guaranteed to be safe to use in HTML contexts,
 * such as, assignment to the innerHTML DOM property, or interpolation into
 * a HTML template in HTML PC_DATA context, in the sense that the use will not
 * result in a Cross-Site-Scripting vulnerability.
 *
 * Instances of this type must be created via the factory methods
 * (`goog.html.SafeHtml.create`, `goog.html.SafeHtml.htmlEscape`),
 * etc and not by invoking its constructor.  The constructor intentionally
 * takes no parameters and the type is immutable; hence only a default instance
 * corresponding to the empty string can be obtained via constructor invocation.
 *
 * Note that there is no `goog.html.SafeHtml.fromConstant`. The reason is that
 * the following code would create an unsafe HTML:
 *
 * ```
 * goog.html.SafeHtml.concat(
 *     goog.html.SafeHtml.fromConstant(goog.string.Const.from('<script>')),
 *     goog.html.SafeHtml.htmlEscape(userInput),
 *     goog.html.SafeHtml.fromConstant(goog.string.Const.from('<\/script>')));
 * ```
 *
 * There's `goog.dom.constHtmlToNode` to create a node from constant strings
 * only.
 *
 * @see goog.html.SafeHtml.create
 * @see goog.html.SafeHtml.htmlEscape
 * @constructor
 * @final
 * @struct
 * @implements {goog.i18n.bidi.DirectionalString}
 * @implements {goog.string.TypedString}
 */
goog.html.SafeHtml = function() {
  /**
   * The contained value of this SafeHtml.  The field has a purposely ugly
   * name to make (non-compiled) code that attempts to directly access this
   * field stand out.
   * @private {!TrustedHTML|string}
   */
  this.privateDoNotAccessOrElseSafeHtmlWrappedValue_ = '';

  /**
   * A type marker used to implement additional run-time type checking.
   * @see goog.html.SafeHtml.unwrap
   * @const {!Object}
   * @private
   */
  this.SAFE_HTML_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ =
      goog.html.SafeHtml.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_;

  /**
   * This SafeHtml's directionality, or null if unknown.
   * @private {?goog.i18n.bidi.Dir}
   */
  this.dir_ = null;
};


/**
 * @define {boolean} Whether to strip out error messages or to leave them in.
 */
goog.html.SafeHtml.ENABLE_ERROR_MESSAGES =
    goog.define('goog.html.SafeHtml.ENABLE_ERROR_MESSAGES', goog.DEBUG);


/**
 * Whether the `style` attribute is supported. Set to false to avoid the byte
 * weight of `goog.html.SafeStyle` where unneeded. An error will be thrown if
 * the `style` attribute is used.
 * @define {boolean}
 */
goog.html.SafeHtml.SUPPORT_STYLE_ATTRIBUTE =
    goog.define('goog.html.SafeHtml.SUPPORT_STYLE_ATTRIBUTE', true);


/**
 * @override
 * @const
 */
goog.html.SafeHtml.prototype.implementsGoogI18nBidiDirectionalString = true;


/** @override */
goog.html.SafeHtml.prototype.getDirection = function() {
  return this.dir_;
};


/**
 * @override
 * @const
 */
goog.html.SafeHtml.prototype.implementsGoogStringTypedString = true;


/**
 * Returns this SafeHtml's value as string.
 *
 * IMPORTANT: In code where it is security relevant that an object's type is
 * indeed `SafeHtml`, use `goog.html.SafeHtml.unwrap` instead of
 * this method. If in doubt, assume that it's security relevant. In particular,
 * note that goog.html functions which return a goog.html type do not guarantee
 * that the returned instance is of the right type. For example:
 *
 * <pre>
 * var fakeSafeHtml = new String('fake');
 * fakeSafeHtml.__proto__ = goog.html.SafeHtml.prototype;
 * var newSafeHtml = goog.html.SafeHtml.htmlEscape(fakeSafeHtml);
 * // newSafeHtml is just an alias for fakeSafeHtml, it's passed through by
 * // goog.html.SafeHtml.htmlEscape() as fakeSafeHtml
 * // instanceof goog.html.SafeHtml.
 * </pre>
 *
 * @see goog.html.SafeHtml.unwrap
 * @override
 */
goog.html.SafeHtml.prototype.getTypedStringValue = function() {
  return this.privateDoNotAccessOrElseSafeHtmlWrappedValue_.toString();
};


if (goog.DEBUG) {
  /**
   * Returns a debug string-representation of this value.
   *
   * To obtain the actual string value wrapped in a SafeHtml, use
   * `goog.html.SafeHtml.unwrap`.
   *
   * @see goog.html.SafeHtml.unwrap
   * @override
   */
  goog.html.SafeHtml.prototype.toString = function() {
    return 'SafeHtml{' + this.privateDoNotAccessOrElseSafeHtmlWrappedValue_ +
        '}';
  };
}


/**
 * Performs a runtime check that the provided object is indeed a SafeHtml
 * object, and returns its value.
 * @param {!goog.html.SafeHtml} safeHtml The object to extract from.
 * @return {string} The SafeHtml object's contained string, unless the run-time
 *     type check fails. In that case, `unwrap` returns an innocuous
 *     string, or, if assertions are enabled, throws
 *     `goog.asserts.AssertionError`.
 */
goog.html.SafeHtml.unwrap = function(safeHtml) {
  return goog.html.SafeHtml.unwrapTrustedHTML(safeHtml).toString();
};


/**
 * Unwraps value as TrustedHTML if supported or as a string if not.
 * @param {!goog.html.SafeHtml} safeHtml
 * @return {!TrustedHTML|string}
 * @see goog.html.SafeHtml.unwrap
 */
goog.html.SafeHtml.unwrapTrustedHTML = function(safeHtml) {
  // Perform additional run-time type-checking to ensure that safeHtml is indeed
  // an instance of the expected type.  This provides some additional protection
  // against security bugs due to application code that disables type checks.
  // Specifically, the following checks are performed:
  // 1. The object is an instance of the expected type.
  // 2. The object is not an instance of a subclass.
  // 3. The object carries a type marker for the expected type. "Faking" an
  // object requires a reference to the type marker, which has names intended
  // to stand out in code reviews.
  if (safeHtml instanceof goog.html.SafeHtml &&
      safeHtml.constructor === goog.html.SafeHtml &&
      safeHtml.SAFE_HTML_TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ ===
          goog.html.SafeHtml.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_) {
    return safeHtml.privateDoNotAccessOrElseSafeHtmlWrappedValue_;
  } else {
    goog.asserts.fail('expected object of type SafeHtml, got \'' +
        safeHtml + '\' of type ' + goog.typeOf(safeHtml));
    return 'type_error:SafeHtml';
  }
};


/**
 * Shorthand for union of types that can sensibly be converted to strings
 * or might already be SafeHtml (as SafeHtml is a goog.string.TypedString).
 * @private
 * @typedef {string|number|boolean|!goog.string.TypedString|
 *           !goog.i18n.bidi.DirectionalString}
 */
goog.html.SafeHtml.TextOrHtml_;


/**
 * Returns HTML-escaped text as a SafeHtml object.
 *
 * If text is of a type that implements
 * `goog.i18n.bidi.DirectionalString`, the directionality of the new
 * `SafeHtml` object is set to `text`'s directionality, if known.
 * Otherwise, the directionality of the resulting SafeHtml is unknown (i.e.,
 * `null`).
 *
 * @param {!goog.html.SafeHtml.TextOrHtml_} textOrHtml The text to escape. If
 *     the parameter is of type SafeHtml it is returned directly (no escaping
 *     is done).
 * @return {!goog.html.SafeHtml} The escaped text, wrapped as a SafeHtml.
 */
goog.html.SafeHtml.htmlEscape = function(textOrHtml) {
  if (textOrHtml instanceof goog.html.SafeHtml) {
    return textOrHtml;
  }
  var textIsObject = typeof textOrHtml == 'object';
  var dir = null;
  if (textIsObject && textOrHtml.implementsGoogI18nBidiDirectionalString) {
    dir = /** @type {!goog.i18n.bidi.DirectionalString} */ (textOrHtml)
              .getDirection();
  }
  var textAsString;
  if (textIsObject && textOrHtml.implementsGoogStringTypedString) {
    textAsString = /** @type {!goog.string.TypedString} */ (textOrHtml)
                       .getTypedStringValue();
  } else {
    textAsString = String(textOrHtml);
  }
  return goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
      goog.string.internal.htmlEscape(textAsString), dir);
};


/**
 * Returns HTML-escaped text as a SafeHtml object, with newlines changed to
 * &lt;br&gt;.
 * @param {!goog.html.SafeHtml.TextOrHtml_} textOrHtml The text to escape. If
 *     the parameter is of type SafeHtml it is returned directly (no escaping
 *     is done).
 * @return {!goog.html.SafeHtml} The escaped text, wrapped as a SafeHtml.
 */
goog.html.SafeHtml.htmlEscapePreservingNewlines = function(textOrHtml) {
  if (textOrHtml instanceof goog.html.SafeHtml) {
    return textOrHtml;
  }
  var html = goog.html.SafeHtml.htmlEscape(textOrHtml);
  return goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
      goog.string.internal.newLineToBr(goog.html.SafeHtml.unwrap(html)),
      html.getDirection());
};


/**
 * Returns HTML-escaped text as a SafeHtml object, with newlines changed to
 * &lt;br&gt; and escaping whitespace to preserve spatial formatting. Character
 * entity #160 is used to make it safer for XML.
 * @param {!goog.html.SafeHtml.TextOrHtml_} textOrHtml The text to escape. If
 *     the parameter is of type SafeHtml it is returned directly (no escaping
 *     is done).
 * @return {!goog.html.SafeHtml} The escaped text, wrapped as a SafeHtml.
 */
goog.html.SafeHtml.htmlEscapePreservingNewlinesAndSpaces = function(
    textOrHtml) {
  if (textOrHtml instanceof goog.html.SafeHtml) {
    return textOrHtml;
  }
  var html = goog.html.SafeHtml.htmlEscape(textOrHtml);
  return goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
      goog.string.internal.whitespaceEscape(goog.html.SafeHtml.unwrap(html)),
      html.getDirection());
};


/**
 * Coerces an arbitrary object into a SafeHtml object.
 *
 * If `textOrHtml` is already of type `goog.html.SafeHtml`, the same
 * object is returned. Otherwise, `textOrHtml` is coerced to string, and
 * HTML-escaped. If `textOrHtml` is of a type that implements
 * `goog.i18n.bidi.DirectionalString`, its directionality, if known, is
 * preserved.
 *
 * @param {!goog.html.SafeHtml.TextOrHtml_} textOrHtml The text or SafeHtml to
 *     coerce.
 * @return {!goog.html.SafeHtml} The resulting SafeHtml object.
 * @deprecated Use goog.html.SafeHtml.htmlEscape.
 */
goog.html.SafeHtml.from = goog.html.SafeHtml.htmlEscape;


/**
 * Converts an arbitrary string into an HTML comment by HTML-escaping the
 * contents and embedding the result between HTML comment markers.
 *
 * Escaping is needed because Internet Explorer supports conditional comments
 * and so may render HTML markup within comments.
 *
 * @param {string} text
 * @return {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.comment = function(text) {
  return goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
      '<!--' + goog.string.internal.htmlEscape(text) + '-->', null);
};


/**
 * @const
 * @private
 */
goog.html.SafeHtml.VALID_NAMES_IN_TAG_ = /^[a-zA-Z0-9-]+$/;


/**
 * Set of attributes containing URL as defined at
 * http://www.w3.org/TR/html5/index.html#attributes-1.
 * @private @const {!Object<string,boolean>}
 */
goog.html.SafeHtml.URL_ATTRIBUTES_ = goog.object.createSet(
    'action', 'cite', 'data', 'formaction', 'href', 'manifest', 'poster',
    'src');


/**
 * Tags which are unsupported via create(). They might be supported via a
 * tag-specific create method. These are tags which might require a
 * TrustedResourceUrl in one of their attributes or a restricted type for
 * their content.
 * @private @const {!Object<string,boolean>}
 */
goog.html.SafeHtml.NOT_ALLOWED_TAG_NAMES_ = goog.object.createSet(
    goog.dom.TagName.APPLET, goog.dom.TagName.BASE, goog.dom.TagName.EMBED,
    goog.dom.TagName.IFRAME, goog.dom.TagName.LINK, goog.dom.TagName.MATH,
    goog.dom.TagName.META, goog.dom.TagName.OBJECT, goog.dom.TagName.SCRIPT,
    goog.dom.TagName.STYLE, goog.dom.TagName.SVG, goog.dom.TagName.TEMPLATE);


/**
 * @typedef {string|number|goog.string.TypedString|
 *     goog.html.SafeStyle.PropertyMap|undefined}
 */
goog.html.SafeHtml.AttributeValue;


/**
 * Creates a SafeHtml content consisting of a tag with optional attributes and
 * optional content.
 *
 * For convenience tag names and attribute names are accepted as regular
 * strings, instead of goog.string.Const. Nevertheless, you should not pass
 * user-controlled values to these parameters. Note that these parameters are
 * syntactically validated at runtime, and invalid values will result in
 * an exception.
 *
 * Example usage:
 *
 * goog.html.SafeHtml.create('br');
 * goog.html.SafeHtml.create('div', {'class': 'a'});
 * goog.html.SafeHtml.create('p', {}, 'a');
 * goog.html.SafeHtml.create('p', {}, goog.html.SafeHtml.create('br'));
 *
 * goog.html.SafeHtml.create('span', {
 *   'style': {'margin': '0'}
 * });
 *
 * To guarantee SafeHtml's type contract is upheld there are restrictions on
 * attribute values and tag names.
 *
 * - For attributes which contain script code (on*), a goog.string.Const is
 *   required.
 * - For attributes which contain style (style), a goog.html.SafeStyle or a
 *   goog.html.SafeStyle.PropertyMap is required.
 * - For attributes which are interpreted as URLs (e.g. src, href) a
 *   goog.html.SafeUrl, goog.string.Const or string is required. If a string
 *   is passed, it will be sanitized with SafeUrl.sanitize().
 * - For tags which can load code or set security relevant page metadata,
 *   more specific goog.html.SafeHtml.create*() functions must be used. Tags
 *   which are not supported by this function are applet, base, embed, iframe,
 *   link, math, object, script, style, svg, and template.
 *
 * @param {!goog.dom.TagName|string} tagName The name of the tag. Only tag names
 *     consisting of [a-zA-Z0-9-] are allowed. Tag names documented above are
 *     disallowed.
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 *     Mapping from attribute names to their values. Only attribute names
 *     consisting of [a-zA-Z0-9-] are allowed. Value of null or undefined causes
 *     the attribute to be omitted.
 * @param {!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>=} opt_content Content to
 *     HTML-escape and put inside the tag. This must be empty for void tags
 *     like <br>. Array elements are concatenated.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 * @throws {Error} If invalid tag name, attribute name, or attribute value is
 *     provided.
 * @throws {goog.asserts.AssertionError} If content for void tag is provided.
 */
goog.html.SafeHtml.create = function(tagName, opt_attributes, opt_content) {
  goog.html.SafeHtml.verifyTagName(String(tagName));
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      String(tagName), opt_attributes, opt_content);
};


/**
 * Verifies if the tag name is valid and if it doesn't change the context.
 * E.g. STRONG is fine but SCRIPT throws because it changes context. See
 * goog.html.SafeHtml.create for an explanation of allowed tags.
 * @param {string} tagName
 * @throws {Error} If invalid tag name is provided.
 * @package
 */
goog.html.SafeHtml.verifyTagName = function(tagName) {
  if (!goog.html.SafeHtml.VALID_NAMES_IN_TAG_.test(tagName)) {
    throw new Error(
        goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
            'Invalid tag name <' + tagName + '>.' :
            '');
  }
  if (tagName.toUpperCase() in goog.html.SafeHtml.NOT_ALLOWED_TAG_NAMES_) {
    throw new Error(
        goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?

            'Tag name <' + tagName + '> is not allowed for SafeHtml.' :
            '');
  }
};


/**
 * Creates a SafeHtml representing an iframe tag.
 *
 * This by default restricts the iframe as much as possible by setting the
 * sandbox attribute to the empty string. If the iframe requires less
 * restrictions, set the sandbox attribute as tight as possible, but do not rely
 * on the sandbox as a security feature because it is not supported by older
 * browsers. If a sandbox is essential to security (e.g. for third-party
 * frames), use createSandboxIframe which checks for browser support.
 *
 * @see https://developer.mozilla.org/en/docs/Web/HTML/Element/iframe#attr-sandbox
 *
 * @param {?goog.html.TrustedResourceUrl=} opt_src The value of the src
 *     attribute. If null or undefined src will not be set.
 * @param {?goog.html.SafeHtml=} opt_srcdoc The value of the srcdoc attribute.
 *     If null or undefined srcdoc will not be set.
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 *     Mapping from attribute names to their values. Only attribute names
 *     consisting of [a-zA-Z0-9-] are allowed. Value of null or undefined causes
 *     the attribute to be omitted.
 * @param {!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>=} opt_content Content to
 *     HTML-escape and put inside the tag. Array elements are concatenated.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 * @throws {Error} If invalid tag name, attribute name, or attribute value is
 *     provided. If opt_attributes contains the src or srcdoc attributes.
 */
goog.html.SafeHtml.createIframe = function(
    opt_src, opt_srcdoc, opt_attributes, opt_content) {
  if (opt_src) {
    // Check whether this is really TrustedResourceUrl.
    goog.html.TrustedResourceUrl.unwrap(opt_src);
  }

  var fixedAttributes = {};
  fixedAttributes['src'] = opt_src || null;
  fixedAttributes['srcdoc'] =
      opt_srcdoc && goog.html.SafeHtml.unwrap(opt_srcdoc);
  var defaultAttributes = {'sandbox': ''};
  var attributes = goog.html.SafeHtml.combineAttributes(
      fixedAttributes, defaultAttributes, opt_attributes);
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      'iframe', attributes, opt_content);
};


/**
 * Creates a SafeHtml representing a sandboxed iframe tag.
 *
 * The sandbox attribute is enforced in its most restrictive mode, an empty
 * string. Consequently, the security requirements for the src and srcdoc
 * attributes are relaxed compared to SafeHtml.createIframe. This function
 * will throw on browsers that do not support the sandbox attribute, as
 * determined by SafeHtml.canUseSandboxIframe.
 *
 * The SafeHtml returned by this function can trigger downloads with no
 * user interaction on Chrome (though only a few, further attempts are blocked).
 * Firefox and IE will block all downloads from the sandbox.
 *
 * @see https://developer.mozilla.org/en/docs/Web/HTML/Element/iframe#attr-sandbox
 * @see https://lists.w3.org/Archives/Public/public-whatwg-archive/2013Feb/0112.html
 *
 * @param {string|!goog.html.SafeUrl=} opt_src The value of the src
 *     attribute. If null or undefined src will not be set.
 * @param {string=} opt_srcdoc The value of the srcdoc attribute.
 *     If null or undefined srcdoc will not be set. Will not be sanitized.
 * @param {!Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 *     Mapping from attribute names to their values. Only attribute names
 *     consisting of [a-zA-Z0-9-] are allowed. Value of null or undefined causes
 *     the attribute to be omitted.
 * @param {!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>=} opt_content Content to
 *     HTML-escape and put inside the tag. Array elements are concatenated.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 * @throws {Error} If invalid tag name, attribute name, or attribute value is
 *     provided. If opt_attributes contains the src, srcdoc or sandbox
 *     attributes. If browser does not support the sandbox attribute on iframe.
 */
goog.html.SafeHtml.createSandboxIframe = function(
    opt_src, opt_srcdoc, opt_attributes, opt_content) {
  if (!goog.html.SafeHtml.canUseSandboxIframe()) {
    throw new Error(
        goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
            'The browser does not support sandboxed iframes.' :
            '');
  }

  var fixedAttributes = {};
  if (opt_src) {
    // Note that sanitize is a no-op on SafeUrl.
    fixedAttributes['src'] =
        goog.html.SafeUrl.unwrap(goog.html.SafeUrl.sanitize(opt_src));
  } else {
    fixedAttributes['src'] = null;
  }
  fixedAttributes['srcdoc'] = opt_srcdoc || null;
  fixedAttributes['sandbox'] = '';
  var attributes =
      goog.html.SafeHtml.combineAttributes(fixedAttributes, {}, opt_attributes);
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      'iframe', attributes, opt_content);
};


/**
 * Checks if the user agent supports sandboxed iframes.
 * @return {boolean}
 */
goog.html.SafeHtml.canUseSandboxIframe = function() {
  return goog.global['HTMLIFrameElement'] &&
      ('sandbox' in goog.global['HTMLIFrameElement'].prototype);
};


/**
 * Creates a SafeHtml representing a script tag with the src attribute.
 * @param {!goog.html.TrustedResourceUrl} src The value of the src
 * attribute.
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=}
 * opt_attributes
 *     Mapping from attribute names to their values. Only attribute names
 *     consisting of [a-zA-Z0-9-] are allowed. Value of null or undefined
 *     causes the attribute to be omitted.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 * @throws {Error} If invalid attribute name or value is provided. If
 *     opt_attributes contains the src attribute.
 */
goog.html.SafeHtml.createScriptSrc = function(src, opt_attributes) {
  // TODO(mlourenco): The charset attribute should probably be blocked. If
  // its value is attacker controlled, the script contains attacker controlled
  // sub-strings (even if properly escaped) and the server does not set charset
  // then XSS is likely possible.
  // https://html.spec.whatwg.org/multipage/scripting.html#dom-script-charset

  // Check whether this is really TrustedResourceUrl.
  goog.html.TrustedResourceUrl.unwrap(src);

  var fixedAttributes = {'src': src};
  var defaultAttributes = {};
  var attributes = goog.html.SafeHtml.combineAttributes(
      fixedAttributes, defaultAttributes, opt_attributes);
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      'script', attributes);
};


/**
 * Creates a SafeHtml representing a script tag. Does not allow the language,
 * src, text or type attributes to be set.
 * @param {!goog.html.SafeScript|!Array<!goog.html.SafeScript>}
 *     script Content to put inside the tag. Array elements are
 *     concatenated.
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 *     Mapping from attribute names to their values. Only attribute names
 *     consisting of [a-zA-Z0-9-] are allowed. Value of null or undefined causes
 *     the attribute to be omitted.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 * @throws {Error} If invalid attribute name or attribute value is provided. If
 *     opt_attributes contains the language, src, text or type attribute.
 */
goog.html.SafeHtml.createScript = function(script, opt_attributes) {
  for (var attr in opt_attributes) {
    var attrLower = attr.toLowerCase();
    if (attrLower == 'language' || attrLower == 'src' || attrLower == 'text' ||
        attrLower == 'type') {
      throw new Error(
          goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
              'Cannot set "' + attrLower + '" attribute' :
              '');
    }
  }

  var content = '';
  script = goog.array.concat(script);
  for (var i = 0; i < script.length; i++) {
    content += goog.html.SafeScript.unwrap(script[i]);
  }
  // Convert to SafeHtml so that it's not HTML-escaped. This is safe because
  // as part of its contract, SafeScript should have no dangerous '<'.
  var htmlContent =
      goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
          content, goog.i18n.bidi.Dir.NEUTRAL);
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      'script', opt_attributes, htmlContent);
};


/**
 * Creates a SafeHtml representing a style tag. The type attribute is set
 * to "text/css".
 * @param {!goog.html.SafeStyleSheet|!Array<!goog.html.SafeStyleSheet>}
 *     styleSheet Content to put inside the tag. Array elements are
 *     concatenated.
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 *     Mapping from attribute names to their values. Only attribute names
 *     consisting of [a-zA-Z0-9-] are allowed. Value of null or undefined causes
 *     the attribute to be omitted.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 * @throws {Error} If invalid attribute name or attribute value is provided. If
 *     opt_attributes contains the type attribute.
 */
goog.html.SafeHtml.createStyle = function(styleSheet, opt_attributes) {
  var fixedAttributes = {'type': 'text/css'};
  var defaultAttributes = {};
  var attributes = goog.html.SafeHtml.combineAttributes(
      fixedAttributes, defaultAttributes, opt_attributes);

  var content = '';
  styleSheet = goog.array.concat(styleSheet);
  for (var i = 0; i < styleSheet.length; i++) {
    content += goog.html.SafeStyleSheet.unwrap(styleSheet[i]);
  }
  // Convert to SafeHtml so that it's not HTML-escaped. This is safe because
  // as part of its contract, SafeStyleSheet should have no dangerous '<'.
  var htmlContent =
      goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
          content, goog.i18n.bidi.Dir.NEUTRAL);
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      'style', attributes, htmlContent);
};


/**
 * Creates a SafeHtml representing a meta refresh tag.
 * @param {!goog.html.SafeUrl|string} url Where to redirect. If a string is
 *     passed, it will be sanitized with SafeUrl.sanitize().
 * @param {number=} opt_secs Number of seconds until the page should be
 *     reloaded. Will be set to 0 if unspecified.
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 */
goog.html.SafeHtml.createMetaRefresh = function(url, opt_secs) {

  // Note that sanitize is a no-op on SafeUrl.
  var unwrappedUrl = goog.html.SafeUrl.unwrap(goog.html.SafeUrl.sanitize(url));

  if (goog.labs.userAgent.browser.isIE() ||
      goog.labs.userAgent.browser.isEdge()) {
    // IE/EDGE can't parse the content attribute if the url contains a
    // semicolon. We can fix this by adding quotes around the url, but then we
    // can't parse quotes in the URL correctly. Also, it seems that IE/EDGE
    // did not unescape semicolons in these URLs at some point in the past. We
    // take a best-effort approach.
    //
    // If the URL has semicolons (which may happen in some cases, see
    // http://www.w3.org/TR/1999/REC-html401-19991224/appendix/notes.html#h-B.2
    // for instance), wrap it in single quotes to protect the semicolons.
    // If the URL has semicolons and single quotes, url-encode the single quotes
    // as well.
    //
    // This is imperfect. Notice that both ' and ; are reserved characters in
    // URIs, so this could do the wrong thing, but at least it will do the wrong
    // thing in only rare cases.
    if (goog.string.internal.contains(unwrappedUrl, ';')) {
      unwrappedUrl = "'" + unwrappedUrl.replace(/'/g, '%27') + "'";
    }
  }
  var attributes = {
    'http-equiv': 'refresh',
    'content': (opt_secs || 0) + '; url=' + unwrappedUrl
  };

  // This function will handle the HTML escaping for attributes.
  return goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse(
      'meta', attributes);
};


/**
 * @param {string} tagName The tag name.
 * @param {string} name The attribute name.
 * @param {!goog.html.SafeHtml.AttributeValue} value The attribute value.
 * @return {string} A "name=value" string.
 * @throws {Error} If attribute value is unsafe for the given tag and attribute.
 * @private
 */
goog.html.SafeHtml.getAttrNameAndValue_ = function(tagName, name, value) {
  // If it's goog.string.Const, allow any valid attribute name.
  if (value instanceof goog.string.Const) {
    value = goog.string.Const.unwrap(value);
  } else if (name.toLowerCase() == 'style') {
    if (goog.html.SafeHtml.SUPPORT_STYLE_ATTRIBUTE) {
      value = goog.html.SafeHtml.getStyleValue_(value);
    } else {
      throw new Error(
          goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
              'Attribute "style" not supported.' :
              '');
    }
  } else if (/^on/i.test(name)) {
    // TODO(jakubvrana): Disallow more attributes with a special meaning.
    throw new Error(
        goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ? 'Attribute "' + name +
                '" requires goog.string.Const value, "' + value + '" given.' :
                                                   '');
    // URL attributes handled differently according to tag.
  } else if (name.toLowerCase() in goog.html.SafeHtml.URL_ATTRIBUTES_) {
    if (value instanceof goog.html.TrustedResourceUrl) {
      value = goog.html.TrustedResourceUrl.unwrap(value);
    } else if (value instanceof goog.html.SafeUrl) {
      value = goog.html.SafeUrl.unwrap(value);
    } else if (typeof value === 'string') {
      value = goog.html.SafeUrl.sanitize(value).getTypedStringValue();
    } else {
      throw new Error(
          goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
              'Attribute "' + name + '" on tag "' + tagName +
                  '" requires goog.html.SafeUrl, goog.string.Const, or' +
                  ' string, value "' + value + '" given.' :
              '');
    }
  }

  // Accept SafeUrl, TrustedResourceUrl, etc. for attributes which only require
  // HTML-escaping.
  if (value.implementsGoogStringTypedString) {
    // Ok to call getTypedStringValue() since there's no reliance on the type
    // contract for security here.
    value =
        /** @type {!goog.string.TypedString} */ (value).getTypedStringValue();
  }

  goog.asserts.assert(
      typeof value === 'string' || typeof value === 'number',
      'String or number value expected, got ' + (typeof value) +
          ' with value: ' + value);
  return name + '="' + goog.string.internal.htmlEscape(String(value)) + '"';
};


/**
 * Gets value allowed in "style" attribute.
 * @param {!goog.html.SafeHtml.AttributeValue} value It could be SafeStyle or a
 *     map which will be passed to goog.html.SafeStyle.create.
 * @return {string} Unwrapped value.
 * @throws {Error} If string value is given.
 * @private
 */
goog.html.SafeHtml.getStyleValue_ = function(value) {
  if (!goog.isObject(value)) {
    throw new Error(
        goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
            'The "style" attribute requires goog.html.SafeStyle or map ' +
                'of style properties, ' + (typeof value) + ' given: ' + value :
            '');
  }
  if (!(value instanceof goog.html.SafeStyle)) {
    // Process the property bag into a style object.
    value = goog.html.SafeStyle.create(value);
  }
  return goog.html.SafeStyle.unwrap(value);
};


/**
 * Creates a SafeHtml content with known directionality consisting of a tag with
 * optional attributes and optional content.
 * @param {!goog.i18n.bidi.Dir} dir Directionality.
 * @param {string} tagName
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 * @param {!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>=} opt_content
 * @return {!goog.html.SafeHtml} The SafeHtml content with the tag.
 */
goog.html.SafeHtml.createWithDir = function(
    dir, tagName, opt_attributes, opt_content) {
  var html = goog.html.SafeHtml.create(tagName, opt_attributes, opt_content);
  html.dir_ = dir;
  return html;
};


/**
 * Creates a new SafeHtml object by joining the parts with separator.
 * @param {!goog.html.SafeHtml.TextOrHtml_} separator
 * @param {!Array<!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>>} parts Parts to join. If a part
 *     contains an array then each member of this array is also joined with the
 *     separator.
 * @return {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.join = function(separator, parts) {
  var separatorHtml = goog.html.SafeHtml.htmlEscape(separator);
  var dir = separatorHtml.getDirection();
  var content = [];

  /**
   * @param {!goog.html.SafeHtml.TextOrHtml_|
   *     !Array<!goog.html.SafeHtml.TextOrHtml_>} argument
   */
  var addArgument = function(argument) {
    if (Array.isArray(argument)) {
      goog.array.forEach(argument, addArgument);
    } else {
      var html = goog.html.SafeHtml.htmlEscape(argument);
      content.push(goog.html.SafeHtml.unwrap(html));
      var htmlDir = html.getDirection();
      if (dir == goog.i18n.bidi.Dir.NEUTRAL) {
        dir = htmlDir;
      } else if (htmlDir != goog.i18n.bidi.Dir.NEUTRAL && dir != htmlDir) {
        dir = null;
      }
    }
  };

  goog.array.forEach(parts, addArgument);
  return goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
      content.join(goog.html.SafeHtml.unwrap(separatorHtml)), dir);
};


/**
 * Creates a new SafeHtml object by concatenating values.
 * @param {...(!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>)} var_args Values to concatenate.
 * @return {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.concat = function(var_args) {
  return goog.html.SafeHtml.join(
      goog.html.SafeHtml.EMPTY, Array.prototype.slice.call(arguments));
};


/**
 * Creates a new SafeHtml object with known directionality by concatenating the
 * values.
 * @param {!goog.i18n.bidi.Dir} dir Directionality.
 * @param {...(!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>)} var_args Elements of array
 *     arguments would be processed recursively.
 * @return {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.concatWithDir = function(dir, var_args) {
  var html = goog.html.SafeHtml.concat(goog.array.slice(arguments, 1));
  html.dir_ = dir;
  return html;
};


/**
 * Type marker for the SafeHtml type, used to implement additional run-time
 * type checking.
 * @const {!Object}
 * @private
 */
goog.html.SafeHtml.TYPE_MARKER_GOOG_HTML_SECURITY_PRIVATE_ = {};


/**
 * Package-internal utility method to create SafeHtml instances.
 *
 * @param {string} html The string to initialize the SafeHtml object with.
 * @param {?goog.i18n.bidi.Dir} dir The directionality of the SafeHtml to be
 *     constructed, or null if unknown.
 * @return {!goog.html.SafeHtml} The initialized SafeHtml object.
 * @package
 */
goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse = function(
    html, dir) {
  return new goog.html.SafeHtml().initSecurityPrivateDoNotAccessOrElse_(
      html, dir);
};


/**
 * Called from createSafeHtmlSecurityPrivateDoNotAccessOrElse(). This
 * method exists only so that the compiler can dead code eliminate static
 * fields (like EMPTY) when they're not accessed.
 * @param {string} html
 * @param {?goog.i18n.bidi.Dir} dir
 * @return {!goog.html.SafeHtml}
 * @private
 */
goog.html.SafeHtml.prototype.initSecurityPrivateDoNotAccessOrElse_ = function(
    html, dir) {
  this.privateDoNotAccessOrElseSafeHtmlWrappedValue_ =
      goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY ?
      goog.html.trustedtypes.PRIVATE_DO_NOT_ACCESS_OR_ELSE_POLICY.createHTML(
          html) :
      html;
  this.dir_ = dir;
  return this;
};


/**
 * Like create() but does not restrict which tags can be constructed.
 *
 * @param {string} tagName Tag name. Set or validated by caller.
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 * @param {(!goog.html.SafeHtml.TextOrHtml_|
 *     !Array<!goog.html.SafeHtml.TextOrHtml_>)=} opt_content
 * @return {!goog.html.SafeHtml}
 * @throws {Error} If invalid or unsafe attribute name or value is provided.
 * @throws {goog.asserts.AssertionError} If content for void tag is provided.
 * @package
 */
goog.html.SafeHtml.createSafeHtmlTagSecurityPrivateDoNotAccessOrElse = function(
    tagName, opt_attributes, opt_content) {
  var dir = null;
  var result = '<' + tagName;
  result += goog.html.SafeHtml.stringifyAttributes(tagName, opt_attributes);

  var content = opt_content;
  if (content == null) {
    content = [];
  } else if (!Array.isArray(content)) {
    content = [content];
  }

  if (goog.dom.tags.isVoidTag(tagName.toLowerCase())) {
    goog.asserts.assert(
        !content.length, 'Void tag <' + tagName + '> does not allow content.');
    result += '>';
  } else {
    var html = goog.html.SafeHtml.concat(content);
    result += '>' + goog.html.SafeHtml.unwrap(html) + '</' + tagName + '>';
    dir = html.getDirection();
  }

  var dirAttribute = opt_attributes && opt_attributes['dir'];
  if (dirAttribute) {
    if (/^(ltr|rtl|auto)$/i.test(dirAttribute)) {
      // If the tag has the "dir" attribute specified then its direction is
      // neutral because it can be safely used in any context.
      dir = goog.i18n.bidi.Dir.NEUTRAL;
    } else {
      dir = null;
    }
  }

  return goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
      result, dir);
};


/**
 * Creates a string with attributes to insert after tagName.
 * @param {string} tagName
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 * @return {string} Returns an empty string if there are no attributes, returns
 *     a string starting with a space otherwise.
 * @throws {Error} If attribute value is unsafe for the given tag and attribute.
 * @package
 */
goog.html.SafeHtml.stringifyAttributes = function(tagName, opt_attributes) {
  var result = '';
  if (opt_attributes) {
    for (var name in opt_attributes) {
      if (!goog.html.SafeHtml.VALID_NAMES_IN_TAG_.test(name)) {
        throw new Error(
            goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
                'Invalid attribute name "' + name + '".' :
                '');
      }
      var value = opt_attributes[name];
      if (value == null) {
        continue;
      }
      result +=
          ' ' + goog.html.SafeHtml.getAttrNameAndValue_(tagName, name, value);
    }
  }
  return result;
};


/**
 * @param {!Object<string, ?goog.html.SafeHtml.AttributeValue>} fixedAttributes
 * @param {!Object<string, string>} defaultAttributes
 * @param {?Object<string, ?goog.html.SafeHtml.AttributeValue>=} opt_attributes
 *     Optional attributes passed to create*().
 * @return {!Object<string, ?goog.html.SafeHtml.AttributeValue>}
 * @throws {Error} If opt_attributes contains an attribute with the same name
 *     as an attribute in fixedAttributes.
 * @package
 */
goog.html.SafeHtml.combineAttributes = function(
    fixedAttributes, defaultAttributes, opt_attributes) {
  var combinedAttributes = {};
  var name;

  for (name in fixedAttributes) {
    goog.asserts.assert(name.toLowerCase() == name, 'Must be lower case');
    combinedAttributes[name] = fixedAttributes[name];
  }
  for (name in defaultAttributes) {
    goog.asserts.assert(name.toLowerCase() == name, 'Must be lower case');
    combinedAttributes[name] = defaultAttributes[name];
  }

  if (opt_attributes) {
    for (name in opt_attributes) {
      var nameLower = name.toLowerCase();
      if (nameLower in fixedAttributes) {
        throw new Error(
            goog.html.SafeHtml.ENABLE_ERROR_MESSAGES ?
                'Cannot override "' + nameLower + '" attribute, got "' + name +
                    '" with value "' + opt_attributes[name] + '"' :
                '');
      }
      if (nameLower in defaultAttributes) {
        delete combinedAttributes[nameLower];
      }
      combinedAttributes[name] = opt_attributes[name];
    }
  }

  return combinedAttributes;
};


/**
 * A SafeHtml instance corresponding to the HTML doctype: "<!DOCTYPE html>".
 * @const {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.DOCTYPE_HTML =
    goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
        '<!DOCTYPE html>', goog.i18n.bidi.Dir.NEUTRAL);


/**
 * A SafeHtml instance corresponding to the empty string.
 * @const {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.EMPTY =
    goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
        '', goog.i18n.bidi.Dir.NEUTRAL);


/**
 * A SafeHtml instance corresponding to the <br> tag.
 * @const {!goog.html.SafeHtml}
 */
goog.html.SafeHtml.BR =
    goog.html.SafeHtml.createSafeHtmlSecurityPrivateDoNotAccessOrElse(
        '<br>', goog.i18n.bidi.Dir.NEUTRAL);

//third_party/javascript/angular/v1_6/goog_hardening/closureSceHelper.js
/**
 * @fileoverview Collection of helpers that link the Angular hardened build to
 *    the Closure goog.html types. This should be compiled alongside Closure,
 *    so that the functions defined here can be called in the hardened Angular
 *    {@code $sce} service.
 *
 * MOE:begin_intracomment_strip
 * @see go/angular-v15-migrate#dependency
 * MOE:end_intracomment_strip
 */

// MOE:begin_strip
// There is no goog.provide to keep JSCompiler from dead-code removing this.
// See http://google3/third_party/java_src/jscomp/java/com/google/javascript/jscomp/JSCompilerRunner.java&l=255&rcl=112285511
// MOE:end_strip

goog.require('goog.html.SafeHtml');
goog.require('goog.html.SafeScript');
goog.require('goog.html.SafeStyle');
goog.require('goog.html.SafeUrl');
goog.require('goog.html.TrustedResourceUrl');


goog.exportSymbol(
    'ng.safehtml.googSceHelper.isGoogHtmlType',

    /**
     * @param {*} value
     * @return {*} true if value is a goog.html type.
     * @public
     */
    function isGoogHtmlType(value) {
      if (value && value.implementsGoogStringTypedString) {
        return true;
      } else {
        return false;
      }
    });

goog.exportSymbol(
    'ng.safehtml.googSceHelper.isCOMPILED',
    /**
     * @return {boolean} the value of COMPILED.
     * @public
     */
    function isCOMPILED() {
      return COMPILED;
    });

goog.exportSymbol(
    'ng.safehtml.googSceHelper.unwrapAny',
    /**
     * Unwraps the value given as a parameter, if it is a goog.html type.
     * Otherwise, returns the parameter. Note that this function will not try to
     * unwrap stock {@code $sce.trustAs} results, and that even though Angular
     * trusts by default  `null`, `undefined` and '', Closure does
     * not, so this function will throw on these values since they are not
     * `goog.html` types.
     * @param {*} value
     * @return {*} An unwrapped version of `value`.
     * @throws {!Error} If the value not of a type among the `goog.html`
     * supported types.
     * @public
     */
    function valueOf(value) {
      if (value instanceof goog.html.TrustedResourceUrl) {
        return goog.html.TrustedResourceUrl.unwrap(value);
      } else if (value instanceof goog.html.SafeHtml) {
        return goog.html.SafeHtml.unwrap(value);
      } else if (value instanceof goog.html.SafeUrl) {
        return goog.html.SafeUrl.unwrap(value);
      } else if (value instanceof goog.html.SafeStyle) {
        return goog.html.SafeStyle.unwrap(value);
      } else if (value instanceof goog.html.SafeScript) {
        return goog.html.SafeScript.unwrap(value);
      } else {
        throw new Error();  // should be caught by Angular
      }
    });

goog.exportSymbol(
    'ng.safehtml.googSceHelper.unwrapGivenContext',
    /**
     * Try to unwrap `maybeTrusted` as a `goog.html` type for {@code
     * type}, then return the value or throw if it failed.
     *
     * @param {string} type Either 'url', 'resourceUrl', 'html', 'js' or 'css'.
     * @param {!goog.html.SafeHtml|!goog.html.SafeUrl|
               !goog.html.TrustedResourceUrl|!goog.html.SafeScript|
               !goog.html.SafeStyle} maybeTrusted Instance of a goog.html type.
     * @return {string} `maybeTrusted` unwrapped to string.
     * @throws {!Error} If maybeTrusted cannot be unwrapped in the given context.
     * @public
     */
    function unwrapGivenContext(type, maybeTrusted) {
      // TODO(rjamet): Is there any good way to get to constants defined
      // in $sce? It seems like we'd need a circular dependency for that.
      if (type == 'html') {  // $sce.HTML
        return goog.html.SafeHtml.unwrap(
            /** @type {!goog.html.SafeHtml} */ (maybeTrusted));
      } else if (type == 'resourceUrl' || type == 'templateUrl') {
        // either $sce.RESOURCE_URL or our new $sce.TEMPLATE_URL
        return goog.html.TrustedResourceUrl.unwrap(
            /** @type {!goog.html.TrustedResourceUrl} */ (maybeTrusted));
      } else if (type == 'url') {  // $sce.URL
        // Angular allows $sce.RESOURCE_URL wherever $sce.URL is allowed.
        if (maybeTrusted instanceof goog.html.TrustedResourceUrl) {
          return goog.html.TrustedResourceUrl.unwrap(maybeTrusted);
        }  // Else assume it's a SafeUrl
        return goog.html.SafeUrl.unwrap(
            /** @type {!goog.html.SafeUrl} */ (maybeTrusted));
      } else if (type == 'css') {  // $sce.CSS
        return goog.html.SafeStyle.unwrap(
            /** @type {!goog.html.SafeStyle} */ (maybeTrusted));
      } else if (type == 'js') {  // $sce.JS
        return goog.html.SafeScript.unwrap(
            /** @type {!goog.html.SafeScript} */ (maybeTrusted));
      }
      throw new Error();  // should be caught by Angular
    });

//third_party/javascript/angular_ui/bootstrap/ui_bootstrap_min_0_13_2.js
/**
 * @license
 * The MIT License
 * 
 * Copyright (c) 2012-2013 the AngularUI Team, https://github.com/organizations/angular-ui/teams/291112
 * 
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 * 
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 * 
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 */
/*
 * @license
 * angular-ui-bootstrap
 * http://angular-ui.github.io/bootstrap/

 * Version: 0.13.2 - 2015-08-05
 * License: MIT
 */
angular.module("ui.bootstrap",["ui.bootstrap.collapse","ui.bootstrap.accordion","ui.bootstrap.alert","ui.bootstrap.bindHtml","ui.bootstrap.buttons","ui.bootstrap.carousel","ui.bootstrap.dateparser","ui.bootstrap.position","ui.bootstrap.datepicker","ui.bootstrap.dropdown","ui.bootstrap.modal","ui.bootstrap.pagination","ui.bootstrap.tooltip","ui.bootstrap.popover","ui.bootstrap.progressbar","ui.bootstrap.rating","ui.bootstrap.tabs","ui.bootstrap.timepicker","ui.bootstrap.transition","ui.bootstrap.typeahead"]),angular.module("ui.bootstrap.collapse",[]).directive("collapse",["$animate",function(a){return{link:function(b,c,d){function e(){c.removeClass("collapse").addClass("collapsing").attr("aria-expanded",!0).attr("aria-hidden",!1),a.addClass(c,"in",{to:{height:c[0].scrollHeight+"px"}}).then(f)}function f(){c.removeClass("collapsing"),c.css({height:"auto"})}function g(){return c.hasClass("collapse")||c.hasClass("in")?(c.css({height:c[0].scrollHeight+"px"}).removeClass("collapse").addClass("collapsing").attr("aria-expanded",!1).attr("aria-hidden",!0),void a.removeClass(c,"in",{to:{height:"0"}}).then(h)):h()}function h(){c.css({height:"0"}),c.removeClass("collapsing"),c.addClass("collapse")}b.$watch(d.collapse,function(a){a?g():e()})}}}]),angular.module("ui.bootstrap.accordion",["ui.bootstrap.collapse"]).constant("accordionConfig",{closeOthers:!0}).controller("AccordionController",["$scope","$attrs","accordionConfig",function(a,b,c){this.groups=[],this.closeOthers=function(d){var e=angular.isDefined(b.closeOthers)?a.$eval(b.closeOthers):c.closeOthers;e&&angular.forEach(this.groups,function(a){a!==d&&(a.isOpen=!1)})},this.addGroup=function(a){var b=this;this.groups.push(a),a.$on("$destroy",function(){b.removeGroup(a)})},this.removeGroup=function(a){var b=this.groups.indexOf(a);-1!==b&&this.groups.splice(b,1)}}]).directive("accordion",function(){return{restrict:"EA",controller:"AccordionController",transclude:!0,replace:!1,templateUrl:"template/accordion/accordion.html"}}).directive("accordionGroup",function(){return{require:"^accordion",restrict:"EA",transclude:!0,replace:!0,templateUrl:"template/accordion/accordion-group.html",scope:{heading:"@",isOpen:"=?",isDisabled:"=?"},controller:function(){this.setHeading=function(a){this.heading=a}},link:function(a,b,c,d){d.addGroup(a),a.$watch("isOpen",function(b){b&&d.closeOthers(a)}),a.toggleOpen=function(){a.isDisabled||(a.isOpen=!a.isOpen)}}}}).directive("accordionHeading",function(){return{restrict:"EA",transclude:!0,template:"",replace:!0,require:"^accordionGroup",link:function(a,b,c,d,e){d.setHeading(e(a,angular.noop))}}}).directive("accordionTransclude",function(){return{require:"^accordionGroup",link:function(a,b,c,d){a.$watch(function(){return d[c.accordionTransclude]},function(a){a&&(b.find("span").html(""),b.find("span").append(a))})}}}),angular.module("ui.bootstrap.alert",[]).controller("AlertController",["$scope","$attrs",function(a,b){a.closeable=!!b.close,this.close=a.close}]).directive("alert",function(){return{restrict:"EA",controller:"AlertController",templateUrl:"template/alert/alert.html",transclude:!0,replace:!0,scope:{type:"@",close:"&"}}}).directive("dismissOnTimeout",["$timeout",function(a){return{require:"alert",link:function(b,c,d,e){a(function(){e.close()},parseInt(d.dismissOnTimeout,10))}}}]),angular.module("ui.bootstrap.bindHtml",[]).value("$bindHtmlUnsafeSuppressDeprecated",!1).directive("bindHtmlUnsafe",["$log","$bindHtmlUnsafeSuppressDeprecated",function(a,b){return function(c,d,e){b||a.warn("bindHtmlUnsafe is now deprecated. Use ngBindHtml instead"),d.addClass("ng-binding").data("$binding",e.bindHtmlUnsafe),c.$watch(e.bindHtmlUnsafe,function(a){d.html(a||"")})}}]),angular.module("ui.bootstrap.buttons",[]).constant("buttonConfig",{activeClass:"active",toggleEvent:"click"}).controller("ButtonsController",["buttonConfig",function(a){this.activeClass=a.activeClass||"active",this.toggleEvent=a.toggleEvent||"click"}]).directive("btnRadio",function(){return{require:["btnRadio","ngModel"],controller:"ButtonsController",link:function(a,b,c,d){var e=d[0],f=d[1];f.$render=function(){b.toggleClass(e.activeClass,angular.equals(f.$modelValue,a.$eval(c.btnRadio)))},b.bind(e.toggleEvent,function(){if(!("disabled"in c)){var d=b.hasClass(e.activeClass);(!d||angular.isDefined(c.uncheckable))&&a.$apply(function(){f.$setViewValue(d?null:a.$eval(c.btnRadio)),f.$render()})}})}}}).directive("btnCheckbox",function(){return{require:["btnCheckbox","ngModel"],controller:"ButtonsController",link:function(a,b,c,d){function e(){return g(c.btnCheckboxTrue,!0)}function f(){return g(c.btnCheckboxFalse,!1)}function g(b,c){var d=a.$eval(b);return angular.isDefined(d)?d:c}var h=d[0],i=d[1];i.$render=function(){b.toggleClass(h.activeClass,angular.equals(i.$modelValue,e()))},b.bind(h.toggleEvent,function(){"disabled"in c||a.$apply(function(){i.$setViewValue(b.hasClass(h.activeClass)?f():e()),i.$render()})})}}}),angular.module("ui.bootstrap.carousel",[]).constant("ANIMATE_CSS",angular.version.minor>=4).controller("CarouselController",["$scope","$element","$interval","$animate","ANIMATE_CSS",function(a,b,c,d,e){function f(b,c,f){r||(angular.extend(b,{direction:f,active:!0}),angular.extend(m.currentSlide||{},{direction:f,active:!1}),d.enabled()&&!a.noTransition&&!a.$currentTransition&&b.$element&&(b.$element.data(p,b.direction),a.$currentTransition=!0,e?d.on("addClass",b.$element,function(b){a.$currentTransition=null,a.$currentTransition||d.off("addClass",b)}):b.$element.one("$animate:close",function(){a.$currentTransition=null})),m.currentSlide=b,q=c,h())}function g(a){if(angular.isUndefined(n[a].index))return n[a];{var b;n.length}for(b=0;b<n.length;++b)if(n[b].index==a)return n[b]}function h(){i();var b=+a.interval;!isNaN(b)&&b>0&&(k=c(j,b))}function i(){k&&(c.cancel(k),k=null)}function j(){var b=+a.interval;l&&!isNaN(b)&&b>0&&n.length?a.next():a.pause()}var k,l,m=this,n=m.slides=a.slides=[],o="uib-noTransition",p="uib-slideDirection",q=-1;m.currentSlide=null;var r=!1;m.select=a.select=function(b,c){var d=m.indexOfSlide(b);void 0===c&&(c=d>m.getCurrentIndex()?"next":"prev"),b&&b!==m.currentSlide&&!a.$currentTransition&&f(b,d,c)},a.$on("$destroy",function(){r=!0}),m.getCurrentIndex=function(){return m.currentSlide&&angular.isDefined(m.currentSlide.index)?+m.currentSlide.index:q},m.indexOfSlide=function(a){return angular.isDefined(a.index)?+a.index:n.indexOf(a)},a.next=function(){var b=(m.getCurrentIndex()+1)%n.length;return 0===b&&a.noWrap()?void a.pause():m.select(g(b),"next")},a.prev=function(){var b=m.getCurrentIndex()-1<0?n.length-1:m.getCurrentIndex()-1;return a.noWrap()&&b===n.length-1?void a.pause():m.select(g(b),"prev")},a.isActive=function(a){return m.currentSlide===a},a.$watch("interval",h),a.$on("$destroy",i),a.play=function(){l||(l=!0,h())},a.pause=function(){a.noPause||(l=!1,i())},m.addSlide=function(b,c){!n.length&&c&&c.data(p,"next"),b.$element=c,n.push(b),1===n.length||b.active?(m.select(n[n.length-1]),1==n.length&&a.play()):b.active=!1},m.removeSlide=function(a){angular.isDefined(a.index)&&n.sort(function(a,b){return+a.index>+b.index});var b=n.indexOf(a);n.splice(b,1),n.length>0&&a.active?m.select(b>=n.length?n[b-1]:n[b]):q>b&&q--,0===n.length&&(m.currentSlide=null)},a.$watch("noTransition",function(a){b.data(o,a)})}]).directive("carousel",[function(){return{restrict:"EA",transclude:!0,replace:!0,controller:"CarouselController",require:"carousel",templateUrl:"template/carousel/carousel.html",scope:{interval:"=",noTransition:"=",noPause:"=",noWrap:"&"}}}]).directive("slide",function(){return{require:"^carousel",restrict:"EA",transclude:!0,replace:!0,templateUrl:"template/carousel/slide.html",scope:{active:"=?",index:"=?"},link:function(a,b,c,d){d.addSlide(a,b),a.$on("$destroy",function(){d.removeSlide(a)}),a.$watch("active",function(b){b&&d.select(a)})}}}).animation(".item",["$injector","$animate","ANIMATE_CSS",function(a,b,c){function d(a,b,c){a.removeClass(b),c&&c()}var e="uib-noTransition",f="uib-slideDirection",g=c?a.get("$animateCss"):null;return{beforeAddClass:function(a,c,h){if("active"==c&&a.parent()&&!a.parent().data(e)){var i=!1,j=a.data(f),k="next"==j?"left":"right",l=d.bind(this,a,k+" "+j,h);return a.addClass(j),g?g(a,{addClass:k}).start().done(l):b.addClass(a,k).then(function(){i||l(),h()}),function(){i=!0}}h()},beforeRemoveClass:function(a,c,h){if("active"===c&&a.parent()&&!a.parent().data(e)){var i=!1,j=a.data(f),k="next"==j?"left":"right",l=d.bind(this,a,k,h);return g?g(a,{addClass:k}).start().done(l):b.addClass(a,k).then(function(){i||l(),h()}),function(){i=!0}}h()}}}]),angular.module("ui.bootstrap.dateparser",[]).service("dateParser",["$log","$locale","orderByFilter",function(a,b,c){function d(a){var b=[],d=a.split("");return angular.forEach(g,function(c,e){var f=a.indexOf(e);if(f>-1){a=a.split(""),d[f]="("+c.regex+")",a[f]="$";for(var g=f+1,h=f+e.length;h>g;g++)d[g]="",a[g]="$";a=a.join(""),b.push({index:f,apply:c.apply})}}),{regex:new RegExp("^"+d.join("")+"$"),map:c(b,"index")}}function e(a,b,c){return 1>c?!1:1===b&&c>28?29===c&&(a%4===0&&a%100!==0||a%400===0):3===b||5===b||8===b||10===b?31>c:!0}var f=/[\\\^\$\*\+\?\|\[\]\(\)\.\{\}]/g;this.parsers={};var g={yyyy:{regex:"\\d{4}",apply:function(a){this.year=+a}},yy:{regex:"\\d{2}",apply:function(a){this.year=+a+2e3}},y:{regex:"\\d{1,4}",apply:function(a){this.year=+a}},MMMM:{regex:b.DATETIME_FORMATS.MONTH.join("|"),apply:function(a){this.month=b.DATETIME_FORMATS.MONTH.indexOf(a)}},MMM:{regex:b.DATETIME_FORMATS.SHORTMONTH.join("|"),apply:function(a){this.month=b.DATETIME_FORMATS.SHORTMONTH.indexOf(a)}},MM:{regex:"0[1-9]|1[0-2]",apply:function(a){this.month=a-1}},M:{regex:"[1-9]|1[0-2]",apply:function(a){this.month=a-1}},dd:{regex:"[0-2][0-9]{1}|3[0-1]{1}",apply:function(a){this.date=+a}},d:{regex:"[1-2]?[0-9]{1}|3[0-1]{1}",apply:function(a){this.date=+a}},EEEE:{regex:b.DATETIME_FORMATS.DAY.join("|")},EEE:{regex:b.DATETIME_FORMATS.SHORTDAY.join("|")},HH:{regex:"(?:0|1)[0-9]|2[0-3]",apply:function(a){this.hours=+a}},H:{regex:"1?[0-9]|2[0-3]",apply:function(a){this.hours=+a}},mm:{regex:"[0-5][0-9]",apply:function(a){this.minutes=+a}},m:{regex:"[0-9]|[1-5][0-9]",apply:function(a){this.minutes=+a}},sss:{regex:"[0-9][0-9][0-9]",apply:function(a){this.milliseconds=+a}},ss:{regex:"[0-5][0-9]",apply:function(a){this.seconds=+a}},s:{regex:"[0-9]|[1-5][0-9]",apply:function(a){this.seconds=+a}}};this.parse=function(c,g,h){if(!angular.isString(c)||!g)return c;g=b.DATETIME_FORMATS[g]||g,g=g.replace(f,"\\$&"),this.parsers[g]||(this.parsers[g]=d(g));var i=this.parsers[g],j=i.regex,k=i.map,l=c.match(j);if(l&&l.length){var m,n;angular.isDate(h)&&!isNaN(h.getTime())?m={year:h.getFullYear(),month:h.getMonth(),date:h.getDate(),hours:h.getHours(),minutes:h.getMinutes(),seconds:h.getSeconds(),milliseconds:h.getMilliseconds()}:(h&&a.warn("dateparser:","baseDate is not a valid date"),m={year:1900,month:0,date:1,hours:0,minutes:0,seconds:0,milliseconds:0});for(var o=1,p=l.length;p>o;o++){var q=k[o-1];q.apply&&q.apply.call(m,l[o])}return e(m.year,m.month,m.date)&&(n=new Date(m.year,m.month,m.date,m.hours,m.minutes,m.seconds,m.milliseconds||0)),n}}}]),angular.module("ui.bootstrap.position",[]).factory("$position",["$document","$window",function(a,b){function c(a,c){return a.currentStyle?a.currentStyle[c]:b.getComputedStyle?b.getComputedStyle(a)[c]:a.style[c]}function d(a){return"static"===(c(a,"position")||"static")}var e=function(b){for(var c=a[0],e=b.offsetParent||c;e&&e!==c&&d(e);)e=e.offsetParent;return e||c};return{position:function(b){var c=this.offset(b),d={top:0,left:0},f=e(b[0]);f!=a[0]&&(d=this.offset(angular.element(f)),d.top+=f.clientTop-f.scrollTop,d.left+=f.clientLeft-f.scrollLeft);var g=b[0].getBoundingClientRect();return{width:g.width||b.prop("offsetWidth"),height:g.height||b.prop("offsetHeight"),top:c.top-d.top,left:c.left-d.left}},offset:function(c){var d=c[0].getBoundingClientRect();return{width:d.width||c.prop("offsetWidth"),height:d.height||c.prop("offsetHeight"),top:d.top+(b.pageYOffset||a[0].documentElement.scrollTop),left:d.left+(b.pageXOffset||a[0].documentElement.scrollLeft)}},positionElements:function(a,b,c,d){var e,f,g,h,i=c.split("-"),j=i[0],k=i[1]||"center";e=d?this.offset(a):this.position(a),f=b.prop("offsetWidth"),g=b.prop("offsetHeight");var l={center:function(){return e.left+e.width/2-f/2},left:function(){return e.left},right:function(){return e.left+e.width}},m={center:function(){return e.top+e.height/2-g/2},top:function(){return e.top},bottom:function(){return e.top+e.height}};switch(j){case"right":h={top:m[k](),left:l[j]()};break;case"left":h={top:m[k](),left:e.left-f};break;case"bottom":h={top:m[j](),left:l[k]()};break;default:h={top:e.top-g,left:l[k]()}}return h}}}]),angular.module("ui.bootstrap.datepicker",["ui.bootstrap.dateparser","ui.bootstrap.position"]).constant("datepickerConfig",{formatDay:"dd",formatMonth:"MMMM",formatYear:"yyyy",formatDayHeader:"EEE",formatDayTitle:"MMMM yyyy",formatMonthTitle:"yyyy",datepickerMode:"day",minMode:"day",maxMode:"year",showWeeks:!0,startingDay:0,yearRange:20,minDate:null,maxDate:null,shortcutPropagation:!1}).controller("DatepickerController",["$scope","$attrs","$parse","$interpolate","$log","dateFilter","datepickerConfig",function(a,b,c,d,e,f,g){var h=this,i={$setViewValue:angular.noop};this.modes=["day","month","year"],angular.forEach(["formatDay","formatMonth","formatYear","formatDayHeader","formatDayTitle","formatMonthTitle","minMode","maxMode","showWeeks","startingDay","yearRange","shortcutPropagation"],function(c,e){h[c]=angular.isDefined(b[c])?8>e?d(b[c])(a.$parent):a.$parent.$eval(b[c]):g[c]}),angular.forEach(["minDate","maxDate"],function(d){b[d]?a.$parent.$watch(c(b[d]),function(a){h[d]=a?new Date(a):null,h.refreshView()}):h[d]=g[d]?new Date(g[d]):null}),a.datepickerMode=a.datepickerMode||g.datepickerMode,a.maxMode=h.maxMode,a.uniqueId="datepicker-"+a.$id+"-"+Math.floor(1e4*Math.random()),angular.isDefined(b.initDate)?(this.activeDate=a.$parent.$eval(b.initDate)||new Date,a.$parent.$watch(b.initDate,function(a){a&&(i.$isEmpty(i.$modelValue)||i.$invalid)&&(h.activeDate=a,h.refreshView())})):this.activeDate=new Date,a.isActive=function(b){return 0===h.compare(b.date,h.activeDate)?(a.activeDateId=b.uid,!0):!1},this.init=function(a){i=a,i.$render=function(){h.render()}},this.render=function(){if(i.$viewValue){var a=new Date(i.$viewValue),b=!isNaN(a);b?this.activeDate=a:e.error('Datepicker directive: "ng-model" value must be a Date object, a number of milliseconds since 01.01.1970 or a string representing an RFC2822 or ISO 8601 date.')}this.refreshView()},this.refreshView=function(){if(this.element){this._refreshView();var a=i.$viewValue?new Date(i.$viewValue):null;i.$setValidity("date-disabled",!a||this.element&&!this.isDisabled(a))}},this.createDateObject=function(a,b){var c=i.$viewValue?new Date(i.$viewValue):null;return{date:a,label:f(a,b),selected:c&&0===this.compare(a,c),disabled:this.isDisabled(a),current:0===this.compare(a,new Date),customClass:this.customClass(a)}},this.isDisabled=function(c){return this.minDate&&this.compare(c,this.minDate)<0||this.maxDate&&this.compare(c,this.maxDate)>0||b.dateDisabled&&a.dateDisabled({date:c,mode:a.datepickerMode})},this.customClass=function(b){return a.customClass({date:b,mode:a.datepickerMode})},this.split=function(a,b){for(var c=[];a.length>0;)c.push(a.splice(0,b));return c},this.fixTimeZone=function(a){var b=a.getHours();a.setHours(23===b?b+2:0)},a.select=function(b){if(a.datepickerMode===h.minMode){var c=i.$viewValue?new Date(i.$viewValue):new Date(0,0,0,0,0,0,0);c.setFullYear(b.getFullYear(),b.getMonth(),b.getDate()),i.$setViewValue(c),i.$render()}else h.activeDate=b,a.datepickerMode=h.modes[h.modes.indexOf(a.datepickerMode)-1]},a.move=function(a){var b=h.activeDate.getFullYear()+a*(h.step.years||0),c=h.activeDate.getMonth()+a*(h.step.months||0);h.activeDate.setFullYear(b,c,1),h.refreshView()},a.toggleMode=function(b){b=b||1,a.datepickerMode===h.maxMode&&1===b||a.datepickerMode===h.minMode&&-1===b||(a.datepickerMode=h.modes[h.modes.indexOf(a.datepickerMode)+b])},a.keys={13:"enter",32:"space",33:"pageup",34:"pagedown",35:"end",36:"home",37:"left",38:"up",39:"right",40:"down"};var j=function(){h.element[0].focus()};a.$on("datepicker.focus",j),a.keydown=function(b){var c=a.keys[b.which];if(c&&!b.shiftKey&&!b.altKey)if(b.preventDefault(),h.shortcutPropagation||b.stopPropagation(),"enter"===c||"space"===c){if(h.isDisabled(h.activeDate))return;a.select(h.activeDate),j()}else!b.ctrlKey||"up"!==c&&"down"!==c?(h.handleKeyDown(c,b),h.refreshView()):(a.toggleMode("up"===c?1:-1),j())}}]).directive("datepicker",function(){return{restrict:"EA",replace:!0,templateUrl:"template/datepicker/datepicker.html",scope:{datepickerMode:"=?",dateDisabled:"&",customClass:"&",shortcutPropagation:"&?"},require:["datepicker","?^ngModel"],controller:"DatepickerController",link:function(a,b,c,d){var e=d[0],f=d[1];f&&e.init(f)}}}).directive("daypicker",["dateFilter",function(a){return{restrict:"EA",replace:!0,templateUrl:"template/datepicker/day.html",require:"^datepicker",link:function(b,c,d,e){function f(a,b){return 1!==b||a%4!==0||a%100===0&&a%400!==0?i[b]:29}function g(a,b){for(var c,d=new Array(b),f=new Date(a),g=0;b>g;)c=new Date(f),e.fixTimeZone(c),d[g++]=c,f.setDate(f.getDate()+1);return d}function h(a){var b=new Date(a);b.setDate(b.getDate()+4-(b.getDay()||7));var c=b.getTime();return b.setMonth(0),b.setDate(1),Math.floor(Math.round((c-b)/864e5)/7)+1}b.showWeeks=e.showWeeks,e.step={months:1},e.element=c;var i=[31,28,31,30,31,30,31,31,30,31,30,31];e._refreshView=function(){var c=e.activeDate.getFullYear(),d=e.activeDate.getMonth(),f=new Date(c,d,1),i=e.startingDay-f.getDay(),j=i>0?7-i:-i,k=new Date(f);j>0&&k.setDate(-j+1);for(var l=g(k,42),m=0;42>m;m++)l[m]=angular.extend(e.createDateObject(l[m],e.formatDay),{secondary:l[m].getMonth()!==d,uid:b.uniqueId+"-"+m});b.labels=new Array(7);for(var n=0;7>n;n++)b.labels[n]={abbr:a(l[n].date,e.formatDayHeader),full:a(l[n].date,"EEEE")};if(b.title=a(e.activeDate,e.formatDayTitle),b.rows=e.split(l,7),b.showWeeks){b.weekNumbers=[];for(var o=(11-e.startingDay)%7,p=b.rows.length,q=0;p>q;q++)b.weekNumbers.push(h(b.rows[q][o].date))}},e.compare=function(a,b){return new Date(a.getFullYear(),a.getMonth(),a.getDate())-new Date(b.getFullYear(),b.getMonth(),b.getDate())},e.handleKeyDown=function(a){var b=e.activeDate.getDate();if("left"===a)b-=1;else if("up"===a)b-=7;else if("right"===a)b+=1;else if("down"===a)b+=7;else if("pageup"===a||"pagedown"===a){var c=e.activeDate.getMonth()+("pageup"===a?-1:1);e.activeDate.setMonth(c,1),b=Math.min(f(e.activeDate.getFullYear(),e.activeDate.getMonth()),b)}else"home"===a?b=1:"end"===a&&(b=f(e.activeDate.getFullYear(),e.activeDate.getMonth()));e.activeDate.setDate(b)},e.refreshView()}}}]).directive("monthpicker",["dateFilter",function(a){return{restrict:"EA",replace:!0,templateUrl:"template/datepicker/month.html",require:"^datepicker",link:function(b,c,d,e){e.step={years:1},e.element=c,e._refreshView=function(){for(var c,d=new Array(12),f=e.activeDate.getFullYear(),g=0;12>g;g++)c=new Date(f,g,1),e.fixTimeZone(c),d[g]=angular.extend(e.createDateObject(c,e.formatMonth),{uid:b.uniqueId+"-"+g});b.title=a(e.activeDate,e.formatMonthTitle),b.rows=e.split(d,3)},e.compare=function(a,b){return new Date(a.getFullYear(),a.getMonth())-new Date(b.getFullYear(),b.getMonth())},e.handleKeyDown=function(a){var b=e.activeDate.getMonth();if("left"===a)b-=1;else if("up"===a)b-=3;else if("right"===a)b+=1;else if("down"===a)b+=3;else if("pageup"===a||"pagedown"===a){var c=e.activeDate.getFullYear()+("pageup"===a?-1:1);e.activeDate.setFullYear(c)}else"home"===a?b=0:"end"===a&&(b=11);e.activeDate.setMonth(b)},e.refreshView()}}}]).directive("yearpicker",["dateFilter",function(){return{restrict:"EA",replace:!0,templateUrl:"template/datepicker/year.html",require:"^datepicker",link:function(a,b,c,d){function e(a){return parseInt((a-1)/f,10)*f+1}var f=d.yearRange;d.step={years:f},d.element=b,d._refreshView=function(){for(var b,c=new Array(f),g=0,h=e(d.activeDate.getFullYear());f>g;g++)b=new Date(h+g,0,1),d.fixTimeZone(b),c[g]=angular.extend(d.createDateObject(b,d.formatYear),{uid:a.uniqueId+"-"+g});a.title=[c[0].label,c[f-1].label].join(" - "),a.rows=d.split(c,5)},d.compare=function(a,b){return a.getFullYear()-b.getFullYear()},d.handleKeyDown=function(a){var b=d.activeDate.getFullYear();"left"===a?b-=1:"up"===a?b-=5:"right"===a?b+=1:"down"===a?b+=5:"pageup"===a||"pagedown"===a?b+=("pageup"===a?-1:1)*d.step.years:"home"===a?b=e(d.activeDate.getFullYear()):"end"===a&&(b=e(d.activeDate.getFullYear())+f-1),d.activeDate.setFullYear(b)},d.refreshView()}}}]).constant("datepickerPopupConfig",{datepickerPopup:"yyyy-MM-dd",html5Types:{date:"yyyy-MM-dd","datetime-local":"yyyy-MM-ddTHH:mm:ss.sss",month:"yyyy-MM"},currentText:"Today",clearText:"Clear",closeText:"Done",closeOnDateSelection:!0,appendToBody:!1,showButtonBar:!0}).directive("datepickerPopup",["$compile","$parse","$document","$position","dateFilter","dateParser","datepickerPopupConfig","$timeout",function(a,b,c,d,e,f,g,h){return{restrict:"EA",require:"ngModel",scope:{isOpen:"=?",currentText:"@",clearText:"@",closeText:"@",dateDisabled:"&",customClass:"&"},link:function(i,j,k,l){function m(a){return a.replace(/([A-Z])/g,function(a){return"-"+a.toLowerCase()})}function n(a){if(angular.isNumber(a)&&(a=new Date(a)),a){if(angular.isDate(a)&&!isNaN(a))return a;if(angular.isString(a)){var b=f.parse(a,p,i.date);return isNaN(b)?void 0:b}return void 0}return null}function o(a,b){var c=a||b;if(!k.ngRequired&&!c)return!0;if(angular.isNumber(c)&&(c=new Date(c)),c){if(angular.isDate(c)&&!isNaN(c))return!0;if(angular.isString(c)){var d=f.parse(c,p);return!isNaN(d)}return!1}return!0}var p,q=angular.isDefined(k.closeOnDateSelection)?i.$parent.$eval(k.closeOnDateSelection):g.closeOnDateSelection,r=angular.isDefined(k.datepickerAppendToBody)?i.$parent.$eval(k.datepickerAppendToBody):g.appendToBody;i.showButtonBar=angular.isDefined(k.showButtonBar)?i.$parent.$eval(k.showButtonBar):g.showButtonBar,i.getText=function(a){return i[a+"Text"]||g[a+"Text"]};var s=!1;if(g.html5Types[k.type]?(p=g.html5Types[k.type],s=!0):(p=k.datepickerPopup||g.datepickerPopup,k.$observe("datepickerPopup",function(a){var b=a||g.datepickerPopup;if(b!==p&&(p=b,l.$modelValue=null,!p))throw new Error("datepickerPopup must have a date format specified.")})),!p)throw new Error("datepickerPopup must have a date format specified.");if(s&&k.datepickerPopup)throw new Error("HTML5 date input types do not support custom formats.");var t=angular.element("<div datepicker-popup-wrap><div datepicker></div></div>");t.attr({"ng-model":"date","ng-change":"dateSelection(date)"});var u=angular.element(t.children()[0]);if(s&&"month"==k.type&&(u.attr("datepicker-mode",'"month"'),u.attr("min-mode","month")),k.datepickerOptions){var v=i.$parent.$eval(k.datepickerOptions);v&&v.initDate&&(i.initDate=v.initDate,u.attr("init-date","initDate"),delete v.initDate),angular.forEach(v,function(a,b){u.attr(m(b),a)})}i.watchData={},angular.forEach(["minDate","maxDate","datepickerMode","initDate","shortcutPropagation"],function(a){if(k[a]){var c=b(k[a]);if(i.$parent.$watch(c,function(b){i.watchData[a]=b}),u.attr(m(a),"watchData."+a),"datepickerMode"===a){var d=c.assign;i.$watch("watchData."+a,function(a,b){angular.isFunction(d)&&a!==b&&d(i.$parent,a)})}}}),k.dateDisabled&&u.attr("date-disabled","dateDisabled({ date: date, mode: mode })"),k.showWeeks&&u.attr("show-weeks",k.showWeeks),k.customClass&&u.attr("custom-class","customClass({ date: date, mode: mode })"),s?l.$formatters.push(function(a){return i.date=a,a}):(l.$$parserName="date",l.$validators.date=o,l.$parsers.unshift(n),l.$formatters.push(function(a){return i.date=a,l.$isEmpty(a)?a:e(a,p)})),i.dateSelection=function(a){angular.isDefined(a)&&(i.date=a);var b=i.date?e(i.date,p):null;j.val(b),l.$setViewValue(b),q&&(i.isOpen=!1,j[0].focus())},l.$viewChangeListeners.push(function(){i.date=f.parse(l.$viewValue,p,i.date)});var w=function(a){i.isOpen&&!j[0].contains(a.target)&&i.$apply(function(){i.isOpen=!1})},x=function(a){27===a.which&&i.isOpen?(a.preventDefault(),a.stopPropagation(),i.$apply(function(){i.isOpen=!1}),j[0].focus()):40!==a.which||i.isOpen||(a.preventDefault(),a.stopPropagation(),i.$apply(function(){i.isOpen=!0}))};j.bind("keydown",x),i.keydown=function(a){27===a.which&&(i.isOpen=!1,j[0].focus())},i.$watch("isOpen",function(a){a?(i.position=r?d.offset(j):d.position(j),i.position.top=i.position.top+j.prop("offsetHeight"),h(function(){i.$broadcast("datepicker.focus"),c.bind("click",w)},0,!1)):c.unbind("click",w)}),i.select=function(a){if("today"===a){var b=new Date;angular.isDate(i.date)?(a=new Date(i.date),a.setFullYear(b.getFullYear(),b.getMonth(),b.getDate())):a=new Date(b.setHours(0,0,0,0))}i.dateSelection(a)},i.close=function(){i.isOpen=!1,j[0].focus()};var y=a(t)(i);t.remove(),r?c.find("body").append(y):j.after(y),i.$on("$destroy",function(){i.isOpen===!0&&i.$apply(function(){i.isOpen=!1}),y.remove(),j.unbind("keydown",x),c.unbind("click",w)})}}}]).directive("datepickerPopupWrap",function(){return{restrict:"EA",replace:!0,transclude:!0,templateUrl:"template/datepicker/popup.html"}}),angular.module("ui.bootstrap.dropdown",["ui.bootstrap.position"]).constant("dropdownConfig",{openClass:"open"}).service("dropdownService",["$document","$rootScope",function(a,b){var c=null;this.open=function(b){c||(a.bind("click",d),a.bind("keydown",e)),c&&c!==b&&(c.isOpen=!1),c=b},this.close=function(b){c===b&&(c=null,a.unbind("click",d),a.unbind("keydown",e))};var d=function(a){if(c&&(!a||"disabled"!==c.getAutoClose())){var d=c.getToggleElement();if(!(a&&d&&d[0].contains(a.target))){var e=c.getDropdownElement();a&&"outsideClick"===c.getAutoClose()&&e&&e[0].contains(a.target)||(c.isOpen=!1,b.$$phase||c.$apply())}}},e=function(a){27===a.which?(c.focusToggleElement(),d()):c.isKeynavEnabled()&&/(38|40)/.test(a.which)&&c.isOpen&&(a.preventDefault(),a.stopPropagation(),c.focusDropdownEntry(a.which))}}]).controller("DropdownController",["$scope","$attrs","$parse","dropdownConfig","dropdownService","$animate","$position","$document","$compile","$templateRequest",function(a,b,c,d,e,f,g,h,i,j){var k,l,m=this,n=a.$new(),o=d.openClass,p=angular.noop,q=b.onToggle?c(b.onToggle):angular.noop,r=!1,s=!1;this.init=function(d){m.$element=d,b.isOpen&&(l=c(b.isOpen),p=l.assign,a.$watch(l,function(a){n.isOpen=!!a})),r=angular.isDefined(b.dropdownAppendToBody),s=angular.isDefined(b.keyboardNav),r&&m.dropdownMenu&&(h.find("body").append(m.dropdownMenu),d.on("$destroy",function(){m.dropdownMenu.remove()}))},this.toggle=function(a){return n.isOpen=arguments.length?!!a:!n.isOpen},this.isOpen=function(){return n.isOpen},n.getToggleElement=function(){return m.toggleElement},n.getAutoClose=function(){return b.autoClose||"always"},n.getElement=function(){return m.$element},n.isKeynavEnabled=function(){return s},n.focusDropdownEntry=function(a){var b=m.dropdownMenu?angular.element(m.dropdownMenu).find("a"):angular.element(m.$element).find("ul").eq(0).find("a");switch(a){case 40:m.selectedOption=angular.isNumber(m.selectedOption)?m.selectedOption===b.length-1?m.selectedOption:m.selectedOption+1:0;break;case 38:if(!angular.isNumber(m.selectedOption))return;m.selectedOption=0===m.selectedOption?0:m.selectedOption-1}b[m.selectedOption].focus()},n.getDropdownElement=function(){return m.dropdownMenu},n.focusToggleElement=function(){m.toggleElement&&m.toggleElement[0].focus()},n.$watch("isOpen",function(b,c){if(r&&m.dropdownMenu){var d=g.positionElements(m.$element,m.dropdownMenu,"bottom-left",!0),h={top:d.top+"px",display:b?"block":"none"},l=m.dropdownMenu.hasClass("dropdown-menu-right");l?(h.left="auto",h.right=window.innerWidth-(d.left+m.$element.prop("offsetWidth"))+"px"):(h.left=d.left+"px",h.right="auto"),m.dropdownMenu.css(h)}if(f[b?"addClass":"removeClass"](m.$element,o).then(function(){angular.isDefined(b)&&b!==c&&q(a,{open:!!b})}),b)m.dropdownMenuTemplateUrl&&j(m.dropdownMenuTemplateUrl).then(function(a){k=n.$new(),i(a.trim())(k,function(a){var b=a;m.dropdownMenu.replaceWith(b),m.dropdownMenu=b})}),n.focusToggleElement(),e.open(n);else{if(m.dropdownMenuTemplateUrl){k&&k.$destroy();var s=angular.element('<ul class="dropdown-menu"></ul>');m.dropdownMenu.replaceWith(s),m.dropdownMenu=s}e.close(n),m.selectedOption=null}angular.isFunction(p)&&p(a,b)}),a.$on("$locationChangeSuccess",function(){"disabled"!==n.getAutoClose()&&(n.isOpen=!1)}),a.$on("$destroy",function(){n.$destroy()})}]).directive("dropdown",function(){return{controller:"DropdownController",link:function(a,b,c,d){d.init(b),b.addClass("dropdown")}}}).directive("dropdownMenu",function(){return{restrict:"AC",require:"?^dropdown",link:function(a,b,c,d){if(d){var e=c.templateUrl;e&&(d.dropdownMenuTemplateUrl=e),d.dropdownMenu||(d.dropdownMenu=b)}}}}).directive("keyboardNav",function(){return{restrict:"A",require:"?^dropdown",link:function(a,b,c,d){b.bind("keydown",function(a){if(-1!==[38,40].indexOf(a.which)){a.preventDefault(),a.stopPropagation();var c=angular.element(b).find("a");switch(a.keyCode){case 40:d.selectedOption=angular.isNumber(d.selectedOption)?d.selectedOption===c.length-1?d.selectedOption:d.selectedOption+1:0;break;case 38:d.selectedOption=0===d.selectedOption?0:d.selectedOption-1}c[d.selectedOption].focus()}})}}}).directive("dropdownToggle",function(){return{require:"?^dropdown",link:function(a,b,c,d){if(d){b.addClass("dropdown-toggle"),d.toggleElement=b;var e=function(e){e.preventDefault(),b.hasClass("disabled")||c.disabled||a.$apply(function(){d.toggle()})};b.bind("click",e),b.attr({"aria-haspopup":!0,"aria-expanded":!1}),a.$watch(d.isOpen,function(a){b.attr("aria-expanded",!!a)}),a.$on("$destroy",function(){b.unbind("click",e)})}}}}),angular.module("ui.bootstrap.modal",[]).factory("$$stackedMap",function(){return{createNew:function(){var a=[];return{add:function(b,c){a.push({key:b,value:c})},get:function(b){for(var c=0;c<a.length;c++)if(b==a[c].key)return a[c]},keys:function(){for(var b=[],c=0;c<a.length;c++)b.push(a[c].key);return b},top:function(){return a[a.length-1]},remove:function(b){for(var c=-1,d=0;d<a.length;d++)if(b==a[d].key){c=d;break}return a.splice(c,1)[0]},removeTop:function(){return a.splice(a.length-1,1)[0]},length:function(){return a.length}}}}}).directive("modalBackdrop",["$animate","$modalStack",function(a,b){function c(c,d,e){e.modalInClass&&(a.addClass(d,e.modalInClass),c.$on(b.NOW_CLOSING_EVENT,function(b,c){var f=c();a.removeClass(d,e.modalInClass).then(f)}))}return{restrict:"EA",replace:!0,templateUrl:"template/modal/backdrop.html",compile:function(a,b){return a.addClass(b.backdropClass),c}}}]).directive("modalWindow",["$modalStack","$q","$animate",function(a,b,c){return{restrict:"EA",scope:{index:"@"},replace:!0,transclude:!0,templateUrl:function(a,b){return b.templateUrl||"template/modal/window.html"},link:function(d,e,f){e.addClass(f.windowClass||""),d.size=f.size,d.close=function(b){var c=a.getTop();c&&c.value.backdrop&&"static"!=c.value.backdrop&&b.target===b.currentTarget&&(b.preventDefault(),b.stopPropagation(),a.dismiss(c.key,"backdrop click"))},d.$isRendered=!0;var g=b.defer();f.$observe("modalRender",function(a){"true"==a&&g.resolve()}),g.promise.then(function(){f.modalInClass&&(c.addClass(e,f.modalInClass),d.$on(a.NOW_CLOSING_EVENT,function(a,b){var d=b();c.removeClass(e,f.modalInClass).then(d)}));var b=e[0].querySelectorAll("[autofocus]");b.length?b[0].focus():e[0].focus();var g=a.getTop();g&&a.modalRendered(g.key)})}}}]).directive("modalAnimationClass",[function(){return{compile:function(a,b){b.modalAnimation&&a.addClass(b.modalAnimationClass)}}}]).directive("modalTransclude",function(){return{link:function(a,b,c,d,e){e(a.$parent,function(a){b.empty(),b.append(a)})}}}).factory("$modalStack",["$animate","$timeout","$document","$compile","$rootScope","$q","$$stackedMap",function(a,b,c,d,e,f,g){function h(){for(var a=-1,b=q.keys(),c=0;c<b.length;c++)q.get(b[c]).value.backdrop&&(a=c);

return a}function i(a,b){var d=c.find("body").eq(0),e=q.get(a).value;q.remove(a),k(e.modalDomEl,e.modalScope,function(){d.toggleClass(p,q.length()>0)}),j(),b&&b.focus?b.focus():d.focus()}function j(){if(m&&-1==h()){var a=n;k(m,n,function(){a=null}),m=void 0,n=void 0}}function k(b,c,d){function e(){e.done||(e.done=!0,a.leave(b),c.$destroy(),d&&d())}var g,h=null,i=function(){return g||(g=f.defer(),h=g.promise),function(){g.resolve()}};return c.$broadcast(r.NOW_CLOSING_EVENT,i),f.when(h).then(e)}function l(a,b,c){return!a.value.modalScope.$broadcast("modal.closing",b,c).defaultPrevented}var m,n,o,p="modal-open",q=g.createNew(),r={NOW_CLOSING_EVENT:"modal.stack.now-closing"},s=0,t="a[href], area[href], input:not([disabled]), button:not([disabled]),select:not([disabled]), textarea:not([disabled]), iframe, object, embed, *[tabindex], *[contenteditable=true]";return e.$watch(h,function(a){n&&(n.index=a)}),c.bind("keydown",function(a){var b=q.top();if(b&&b.value.keyboard)switch(a.which){case 27:a.preventDefault(),e.$apply(function(){r.dismiss(b.key,"escape key press")});break;case 9:r.loadFocusElementList(b);var c=!1;a.shiftKey?r.isFocusInFirstItem(a)&&(c=r.focusLastFocusableElement()):r.isFocusInLastItem(a)&&(c=r.focusFirstFocusableElement()),c&&(a.preventDefault(),a.stopPropagation())}}),r.open=function(a,b){var f=c[0].activeElement;q.add(a,{deferred:b.deferred,renderDeferred:b.renderDeferred,modalScope:b.scope,backdrop:b.backdrop,keyboard:b.keyboard});var g=c.find("body").eq(0),i=h();if(i>=0&&!m){n=e.$new(!0),n.index=i;var j=angular.element('<div modal-backdrop="modal-backdrop"></div>');j.attr("backdrop-class",b.backdropClass),b.animation&&j.attr("modal-animation","true"),m=d(j)(n),g.append(m)}var k=angular.element('<div modal-window="modal-window"></div>');k.attr({"template-url":b.windowTemplateUrl,"window-class":b.windowClass,size:b.size,index:q.length()-1,animate:"animate"}).html(b.content),b.animation&&k.attr("modal-animation","true");var l=d(k)(b.scope);q.top().value.modalDomEl=l,q.top().value.modalOpener=f,g.append(l),g.addClass(p),r.clearFocusListCache()},r.close=function(a,b){var c=q.get(a);return c&&l(c,b,!0)?(c.value.deferred.resolve(b),i(a,c.value.modalOpener),!0):!c},r.dismiss=function(a,b){var c=q.get(a);return c&&l(c,b,!1)?(c.value.deferred.reject(b),i(a,c.value.modalOpener),!0):!c},r.dismissAll=function(a){for(var b=this.getTop();b&&this.dismiss(b.key,a);)b=this.getTop()},r.getTop=function(){return q.top()},r.modalRendered=function(a){var b=q.get(a);b&&b.value.renderDeferred.resolve()},r.focusFirstFocusableElement=function(){return o.length>0?(o[0].focus(),!0):!1},r.focusLastFocusableElement=function(){return o.length>0?(o[o.length-1].focus(),!0):!1},r.isFocusInFirstItem=function(a){return o.length>0?(a.target||a.srcElement)==o[0]:!1},r.isFocusInLastItem=function(a){return o.length>0?(a.target||a.srcElement)==o[o.length-1]:!1},r.clearFocusListCache=function(){o=[],s=0},r.loadFocusElementList=function(a){if((void 0===o||!o.length0)&&a){var b=a.value.modalDomEl;b&&b.length&&(o=b[0].querySelectorAll(t))}},r}]).provider("$modal",function(){var a={options:{animation:!0,backdrop:!0,keyboard:!0},$get:["$injector","$rootScope","$q","$templateRequest","$controller","$modalStack",function(b,c,d,e,f,g){function h(a){return a.template?d.when(a.template):e(angular.isFunction(a.templateUrl)?a.templateUrl():a.templateUrl)}function i(a){var c=[];return angular.forEach(a,function(a){(angular.isFunction(a)||angular.isArray(a))&&c.push(d.when(b.invoke(a)))}),c}var j={};return j.open=function(b){var e=d.defer(),j=d.defer(),k=d.defer(),l={result:e.promise,opened:j.promise,rendered:k.promise,close:function(a){return g.close(l,a)},dismiss:function(a){return g.dismiss(l,a)}};if(b=angular.extend({},a.options,b),b.resolve=b.resolve||{},!b.template&&!b.templateUrl)throw new Error("One of template or templateUrl options is required.");var m=d.all([h(b)].concat(i(b.resolve)));return m.then(function(a){var d=(b.scope||c).$new();d.$close=l.close,d.$dismiss=l.dismiss;var h,i={},j=1;b.controller&&(i.$scope=d,i.$modalInstance=l,angular.forEach(b.resolve,function(b,c){i[c]=a[j++]}),h=f(b.controller,i),b.controllerAs&&(b.bindToController&&angular.extend(h,d),d[b.controllerAs]=h)),g.open(l,{scope:d,deferred:e,renderDeferred:k,content:a[0],animation:b.animation,backdrop:b.backdrop,keyboard:b.keyboard,backdropClass:b.backdropClass,windowClass:b.windowClass,windowTemplateUrl:b.windowTemplateUrl,size:b.size})},function(a){e.reject(a)}),m.then(function(){j.resolve(!0)},function(a){j.reject(a)}),l},j}]};return a}),angular.module("ui.bootstrap.pagination",[]).controller("PaginationController",["$scope","$attrs","$parse",function(a,b,c){var d=this,e={$setViewValue:angular.noop},f=b.numPages?c(b.numPages).assign:angular.noop;this.init=function(g,h){e=g,this.config=h,e.$render=function(){d.render()},b.itemsPerPage?a.$parent.$watch(c(b.itemsPerPage),function(b){d.itemsPerPage=parseInt(b,10),a.totalPages=d.calculateTotalPages()}):this.itemsPerPage=h.itemsPerPage,a.$watch("totalItems",function(){a.totalPages=d.calculateTotalPages()}),a.$watch("totalPages",function(b){f(a.$parent,b),a.page>b?a.selectPage(b):e.$render()})},this.calculateTotalPages=function(){var b=this.itemsPerPage<1?1:Math.ceil(a.totalItems/this.itemsPerPage);return Math.max(b||0,1)},this.render=function(){a.page=parseInt(e.$viewValue,10)||1},a.selectPage=function(b,c){var d=!a.ngDisabled||!c;d&&a.page!==b&&b>0&&b<=a.totalPages&&(c&&c.target&&c.target.blur(),e.$setViewValue(b),e.$render())},a.getText=function(b){return a[b+"Text"]||d.config[b+"Text"]},a.noPrevious=function(){return 1===a.page},a.noNext=function(){return a.page===a.totalPages}}]).constant("paginationConfig",{itemsPerPage:10,boundaryLinks:!1,directionLinks:!0,firstText:"First",previousText:"Previous",nextText:"Next",lastText:"Last",rotate:!0}).directive("pagination",["$parse","paginationConfig",function(a,b){return{restrict:"EA",scope:{totalItems:"=",firstText:"@",previousText:"@",nextText:"@",lastText:"@",ngDisabled:"="},require:["pagination","?ngModel"],controller:"PaginationController",templateUrl:"template/pagination/pagination.html",replace:!0,link:function(c,d,e,f){function g(a,b,c){return{number:a,text:b,active:c}}function h(a,b){var c=[],d=1,e=b,f=angular.isDefined(k)&&b>k;f&&(l?(d=Math.max(a-Math.floor(k/2),1),e=d+k-1,e>b&&(e=b,d=e-k+1)):(d=(Math.ceil(a/k)-1)*k+1,e=Math.min(d+k-1,b)));for(var h=d;e>=h;h++){var i=g(h,h,h===a);c.push(i)}if(f&&!l){if(d>1){var j=g(d-1,"...",!1);c.unshift(j)}if(b>e){var m=g(e+1,"...",!1);c.push(m)}}return c}var i=f[0],j=f[1];if(j){var k=angular.isDefined(e.maxSize)?c.$parent.$eval(e.maxSize):b.maxSize,l=angular.isDefined(e.rotate)?c.$parent.$eval(e.rotate):b.rotate;c.boundaryLinks=angular.isDefined(e.boundaryLinks)?c.$parent.$eval(e.boundaryLinks):b.boundaryLinks,c.directionLinks=angular.isDefined(e.directionLinks)?c.$parent.$eval(e.directionLinks):b.directionLinks,i.init(j,b),e.maxSize&&c.$parent.$watch(a(e.maxSize),function(a){k=parseInt(a,10),i.render()});var m=i.render;i.render=function(){m(),c.page>0&&c.page<=c.totalPages&&(c.pages=h(c.page,c.totalPages))}}}}}]).constant("pagerConfig",{itemsPerPage:10,previousText:"« Previous",nextText:"Next »",align:!0}).directive("pager",["pagerConfig",function(a){return{restrict:"EA",scope:{totalItems:"=",previousText:"@",nextText:"@"},require:["pager","?ngModel"],controller:"PaginationController",templateUrl:"template/pagination/pager.html",replace:!0,link:function(b,c,d,e){var f=e[0],g=e[1];g&&(b.align=angular.isDefined(d.align)?b.$parent.$eval(d.align):a.align,f.init(g,a))}}}]),angular.module("ui.bootstrap.tooltip",["ui.bootstrap.position","ui.bootstrap.bindHtml"]).provider("$tooltip",function(){function a(a){var b=/[A-Z]/g,c="-";return a.replace(b,function(a,b){return(b?c:"")+a.toLowerCase()})}var b={placement:"top",animation:!0,popupDelay:0,useContentExp:!1},c={mouseenter:"mouseleave",click:"click",focus:"blur"},d={};this.options=function(a){angular.extend(d,a)},this.setTriggers=function(a){angular.extend(c,a)},this.$get=["$window","$compile","$timeout","$document","$position","$interpolate",function(e,f,g,h,i,j){return function(e,k,l,m){function n(a){var b=a||m.trigger||l,d=c[b]||b;return{show:b,hide:d}}m=angular.extend({},b,d,m);var o=a(e),p=j.startSymbol(),q=j.endSymbol(),r="<div "+o+'-popup title="'+p+"title"+q+'" '+(m.useContentExp?'content-exp="contentExp()" ':'content="'+p+"content"+q+'" ')+'placement="'+p+"placement"+q+'" popup-class="'+p+"popupClass"+q+'" animation="animation" is-open="isOpen"origin-scope="origScope" ></div>';return{restrict:"EA",compile:function(){var a=f(r);return function(b,c,d){function f(){E.isOpen?l():j()}function j(){(!D||b.$eval(d[k+"Enable"]))&&(s(),E.popupDelay?A||(A=g(o,E.popupDelay,!1),A.then(function(a){a()})):o()())}function l(){b.$apply(function(){p()})}function o(){return A=null,z&&(g.cancel(z),z=null),(m.useContentExp?E.contentExp():E.content)?(q(),x.css({top:0,left:0,display:"block"}),E.$digest(),F(),E.isOpen=!0,E.$apply(),F):angular.noop}function p(){E.isOpen=!1,g.cancel(A),A=null,E.animation?z||(z=g(r,500)):r()}function q(){x&&r(),y=E.$new(),x=a(y,function(a){B?h.find("body").append(a):c.after(a)}),m.useContentExp&&y.$watch("contentExp()",function(a){G(),!a&&E.isOpen&&p()})}function r(){z=null,x&&(x.remove(),x=null),y&&(y.$destroy(),y=null)}function s(){t(),u(),v()}function t(){E.popupClass=d[k+"Class"]}function u(){var a=d[k+"Placement"];E.placement=angular.isDefined(a)?a:m.placement}function v(){var a=d[k+"PopupDelay"],b=parseInt(a,10);E.popupDelay=isNaN(b)?m.popupDelay:b}function w(){var a=d[k+"Trigger"];H(),C=n(a),C.show===C.hide?c.bind(C.show,f):(c.bind(C.show,j),c.bind(C.hide,l))}var x,y,z,A,B=angular.isDefined(m.appendToBody)?m.appendToBody:!1,C=n(void 0),D=angular.isDefined(d[k+"Enable"]),E=b.$new(!0),F=function(){if(x){var a=i.positionElements(c,x,E.placement,B);a.top+="px",a.left+="px",x.css(a)}},G=function(){g(F,0,!1)};E.origScope=b,E.isOpen=!1,E.contentExp=function(){return b.$eval(d[e])},m.useContentExp||d.$observe(e,function(a){E.content=a,G(),!a&&E.isOpen&&p()}),d.$observe("disabled",function(a){a&&E.isOpen&&p()}),d.$observe(k+"Title",function(a){E.title=a,G()}),d.$observe(k+"Placement",function(){E.isOpen&&g(function(){u(),o()()},0,!1)});var H=function(){c.unbind(C.show,j),c.unbind(C.hide,l)};w();var I=b.$eval(d[k+"Animation"]);E.animation=angular.isDefined(I)?!!I:m.animation;var J=b.$eval(d[k+"AppendToBody"]);B=angular.isDefined(J)?J:B,B&&b.$on("$locationChangeSuccess",function(){E.isOpen&&p()}),b.$on("$destroy",function(){g.cancel(z),g.cancel(A),H(),r(),E=null})}}}}}]}).directive("tooltipTemplateTransclude",["$animate","$sce","$compile","$templateRequest",function(a,b,c,d){return{link:function(e,f,g){var h,i,j,k=e.$eval(g.tooltipTemplateTranscludeScope),l=0,m=function(){i&&(i.remove(),i=null),h&&(h.$destroy(),h=null),j&&(a.leave(j).then(function(){i=null}),i=j,j=null)};e.$watch(b.parseAsResourceUrl(g.tooltipTemplateTransclude),function(b){var g=++l;b?(d(b,!0).then(function(d){if(g===l){var e=k.$new(),i=d,n=c(i)(e,function(b){m(),a.enter(b,f)});h=e,j=n,h.$emit("$includeContentLoaded",b)}},function(){g===l&&(m(),e.$emit("$includeContentError",b))}),e.$emit("$includeContentRequested",b)):m()}),e.$on("$destroy",m)}}}]).directive("tooltipClasses",function(){return{restrict:"A",link:function(a,b,c){a.placement&&b.addClass(a.placement),a.popupClass&&b.addClass(a.popupClass),a.animation()&&b.addClass(c.tooltipAnimationClass)}}}).directive("tooltipPopup",function(){return{restrict:"EA",replace:!0,scope:{content:"@",placement:"@",popupClass:"@",animation:"&",isOpen:"&"},templateUrl:"template/tooltip/tooltip-popup.html"}}).directive("tooltip",["$tooltip",function(a){return a("tooltip","tooltip","mouseenter")}]).directive("tooltipTemplatePopup",function(){return{restrict:"EA",replace:!0,scope:{contentExp:"&",placement:"@",popupClass:"@",animation:"&",isOpen:"&",originScope:"&"},templateUrl:"template/tooltip/tooltip-template-popup.html"}}).directive("tooltipTemplate",["$tooltip",function(a){return a("tooltipTemplate","tooltip","mouseenter",{useContentExp:!0})}]).directive("tooltipHtmlPopup",function(){return{restrict:"EA",replace:!0,scope:{contentExp:"&",placement:"@",popupClass:"@",animation:"&",isOpen:"&"},templateUrl:"template/tooltip/tooltip-html-popup.html"}}).directive("tooltipHtml",["$tooltip",function(a){return a("tooltipHtml","tooltip","mouseenter",{useContentExp:!0})}]).directive("tooltipHtmlUnsafePopup",function(){return{restrict:"EA",replace:!0,scope:{content:"@",placement:"@",popupClass:"@",animation:"&",isOpen:"&"},templateUrl:"template/tooltip/tooltip-html-unsafe-popup.html"}}).value("tooltipHtmlUnsafeSuppressDeprecated",!1).directive("tooltipHtmlUnsafe",["$tooltip","tooltipHtmlUnsafeSuppressDeprecated","$log",function(a,b,c){return b||c.warn("tooltip-html-unsafe is now deprecated. Use tooltip-html or tooltip-template instead."),a("tooltipHtmlUnsafe","tooltip","mouseenter")}]),angular.module("ui.bootstrap.popover",["ui.bootstrap.tooltip"]).directive("popoverTemplatePopup",function(){return{restrict:"EA",replace:!0,scope:{title:"@",contentExp:"&",placement:"@",popupClass:"@",animation:"&",isOpen:"&",originScope:"&"},templateUrl:"template/popover/popover-template.html"}}).directive("popoverTemplate",["$tooltip",function(a){return a("popoverTemplate","popover","click",{useContentExp:!0})}]).directive("popoverHtmlPopup",function(){return{restrict:"EA",replace:!0,scope:{contentExp:"&",title:"@",placement:"@",popupClass:"@",animation:"&",isOpen:"&"},templateUrl:"template/popover/popover-html.html"}}).directive("popoverHtml",["$tooltip",function(a){return a("popoverHtml","popover","click",{useContentExp:!0})}]).directive("popoverPopup",function(){return{restrict:"EA",replace:!0,scope:{title:"@",content:"@",placement:"@",popupClass:"@",animation:"&",isOpen:"&"},templateUrl:"template/popover/popover.html"}}).directive("popover",["$tooltip",function(a){return a("popover","popover","click")}]),angular.module("ui.bootstrap.progressbar",[]).constant("progressConfig",{animate:!0,max:100}).controller("ProgressController",["$scope","$attrs","progressConfig",function(a,b,c){var d=this,e=angular.isDefined(b.animate)?a.$parent.$eval(b.animate):c.animate;this.bars=[],a.max=angular.isDefined(a.max)?a.max:c.max,this.addBar=function(b,c){e||c.css({transition:"none"}),this.bars.push(b),b.max=a.max,b.$watch("value",function(){b.recalculatePercentage()}),b.recalculatePercentage=function(){b.percent=+(100*b.value/b.max).toFixed(2);var a=0;d.bars.forEach(function(b){a+=b.percent}),a>100&&(b.percent-=a-100)},b.$on("$destroy",function(){c=null,d.removeBar(b)})},this.removeBar=function(a){this.bars.splice(this.bars.indexOf(a),1)},a.$watch("max",function(){d.bars.forEach(function(b){b.max=a.max,b.recalculatePercentage()})})}]).directive("progress",function(){return{restrict:"EA",replace:!0,transclude:!0,controller:"ProgressController",require:"progress",scope:{max:"=?"},templateUrl:"template/progressbar/progress.html"}}).directive("bar",function(){return{restrict:"EA",replace:!0,transclude:!0,require:"^progress",scope:{value:"=",type:"@"},templateUrl:"template/progressbar/bar.html",link:function(a,b,c,d){d.addBar(a,b)}}}).directive("progressbar",function(){return{restrict:"EA",replace:!0,transclude:!0,controller:"ProgressController",scope:{value:"=",max:"=?",type:"@"},templateUrl:"template/progressbar/progressbar.html",link:function(a,b,c,d){d.addBar(a,angular.element(b.children()[0]))}}}),angular.module("ui.bootstrap.rating",[]).constant("ratingConfig",{max:5,stateOn:null,stateOff:null,titles:["one","two","three","four","five"]}).controller("RatingController",["$scope","$attrs","ratingConfig",function(a,b,c){var d={$setViewValue:angular.noop};this.init=function(e){d=e,d.$render=this.render,d.$formatters.push(function(a){return angular.isNumber(a)&&a<<0!==a&&(a=Math.round(a)),a}),this.stateOn=angular.isDefined(b.stateOn)?a.$parent.$eval(b.stateOn):c.stateOn,this.stateOff=angular.isDefined(b.stateOff)?a.$parent.$eval(b.stateOff):c.stateOff;var f=angular.isDefined(b.titles)?a.$parent.$eval(b.titles):c.titles;this.titles=angular.isArray(f)&&f.length>0?f:c.titles;var g=angular.isDefined(b.ratingStates)?a.$parent.$eval(b.ratingStates):new Array(angular.isDefined(b.max)?a.$parent.$eval(b.max):c.max);a.range=this.buildTemplateObjects(g)},this.buildTemplateObjects=function(a){for(var b=0,c=a.length;c>b;b++)a[b]=angular.extend({index:b},{stateOn:this.stateOn,stateOff:this.stateOff,title:this.getTitle(b)},a[b]);return a},this.getTitle=function(a){return a>=this.titles.length?a+1:this.titles[a]},a.rate=function(b){!a.readonly&&b>=0&&b<=a.range.length&&(d.$setViewValue(d.$viewValue===b?0:b),d.$render())},a.enter=function(b){a.readonly||(a.value=b),a.onHover({value:b})},a.reset=function(){a.value=d.$viewValue,a.onLeave()},a.onKeydown=function(b){/(37|38|39|40)/.test(b.which)&&(b.preventDefault(),b.stopPropagation(),a.rate(a.value+(38===b.which||39===b.which?1:-1)))},this.render=function(){a.value=d.$viewValue}}]).directive("rating",function(){return{restrict:"EA",require:["rating","ngModel"],scope:{readonly:"=?",onHover:"&",onLeave:"&"},controller:"RatingController",templateUrl:"template/rating/rating.html",replace:!0,link:function(a,b,c,d){var e=d[0],f=d[1];e.init(f)}}}),angular.module("ui.bootstrap.tabs",[]).controller("TabsetController",["$scope",function(a){var b=this,c=b.tabs=a.tabs=[];b.select=function(a){angular.forEach(c,function(b){b.active&&b!==a&&(b.active=!1,b.onDeselect())}),a.active=!0,a.onSelect()},b.addTab=function(a){c.push(a),1===c.length&&a.active!==!1?a.active=!0:a.active?b.select(a):a.active=!1},b.removeTab=function(a){var e=c.indexOf(a);if(a.active&&c.length>1&&!d){var f=e==c.length-1?e-1:e+1;b.select(c[f])}c.splice(e,1)};var d;a.$on("$destroy",function(){d=!0})}]).directive("tabset",function(){return{restrict:"EA",transclude:!0,replace:!0,scope:{type:"@"},controller:"TabsetController",templateUrl:"template/tabs/tabset.html",link:function(a,b,c){a.vertical=angular.isDefined(c.vertical)?a.$parent.$eval(c.vertical):!1,a.justified=angular.isDefined(c.justified)?a.$parent.$eval(c.justified):!1}}}).directive("tab",["$parse","$log",function(a,b){return{require:"^tabset",restrict:"EA",replace:!0,templateUrl:"template/tabs/tab.html",transclude:!0,scope:{active:"=?",heading:"@",onSelect:"&select",onDeselect:"&deselect"},controller:function(){},link:function(c,d,e,f,g){c.$watch("active",function(a){a&&f.select(c)}),c.disabled=!1,e.disable&&c.$parent.$watch(a(e.disable),function(a){c.disabled=!!a}),e.disabled&&(b.warn('Use of "disabled" attribute has been deprecated, please use "disable"'),c.$parent.$watch(a(e.disabled),function(a){c.disabled=!!a})),c.select=function(){c.disabled||(c.active=!0)},f.addTab(c),c.$on("$destroy",function(){f.removeTab(c)}),c.$transcludeFn=g}}}]).directive("tabHeadingTransclude",[function(){return{restrict:"A",require:"^tab",link:function(a,b){a.$watch("headingElement",function(a){a&&(b.html(""),b.append(a))})}}}]).directive("tabContentTransclude",function(){function a(a){return a.tagName&&(a.hasAttribute("tab-heading")||a.hasAttribute("data-tab-heading")||"tab-heading"===a.tagName.toLowerCase()||"data-tab-heading"===a.tagName.toLowerCase())}return{restrict:"A",require:"^tabset",link:function(b,c,d){var e=b.$eval(d.tabContentTransclude);e.$transcludeFn(e.$parent,function(b){angular.forEach(b,function(b){a(b)?e.headingElement=b:c.append(b)})})}}}),angular.module("ui.bootstrap.timepicker",[]).constant("timepickerConfig",{hourStep:1,minuteStep:1,showMeridian:!0,meridians:null,readonlyInput:!1,mousewheel:!0,arrowkeys:!0,showSpinners:!0}).controller("TimepickerController",["$scope","$attrs","$parse","$log","$locale","timepickerConfig",function(a,b,c,d,e,f){function g(){var b=parseInt(a.hours,10),c=a.showMeridian?b>0&&13>b:b>=0&&24>b;return c?(a.showMeridian&&(12===b&&(b=0),a.meridian===p[1]&&(b+=12)),b):void 0}function h(){var b=parseInt(a.minutes,10);return b>=0&&60>b?b:void 0}function i(a){return angular.isDefined(a)&&a.toString().length<2?"0"+a:a.toString()}function j(a){k(),o.$setViewValue(new Date(n)),l(a)}function k(){o.$setValidity("time",!0),a.invalidHours=!1,a.invalidMinutes=!1}function l(b){var c=n.getHours(),d=n.getMinutes();a.showMeridian&&(c=0===c||12===c?12:c%12),a.hours="h"===b?c:i(c),"m"!==b&&(a.minutes=i(d)),a.meridian=n.getHours()<12?p[0]:p[1]}function m(a){var b=new Date(n.getTime()+6e4*a);n.setHours(b.getHours(),b.getMinutes()),j()}var n=new Date,o={$setViewValue:angular.noop},p=angular.isDefined(b.meridians)?a.$parent.$eval(b.meridians):f.meridians||e.DATETIME_FORMATS.AMPMS;this.init=function(c,d){o=c,o.$render=this.render,o.$formatters.unshift(function(a){return a?new Date(a):null});var e=d.eq(0),g=d.eq(1),h=angular.isDefined(b.mousewheel)?a.$parent.$eval(b.mousewheel):f.mousewheel;h&&this.setupMousewheelEvents(e,g);var i=angular.isDefined(b.arrowkeys)?a.$parent.$eval(b.arrowkeys):f.arrowkeys;i&&this.setupArrowkeyEvents(e,g),a.readonlyInput=angular.isDefined(b.readonlyInput)?a.$parent.$eval(b.readonlyInput):f.readonlyInput,this.setupInputEvents(e,g)};var q=f.hourStep;b.hourStep&&a.$parent.$watch(c(b.hourStep),function(a){q=parseInt(a,10)});var r=f.minuteStep;b.minuteStep&&a.$parent.$watch(c(b.minuteStep),function(a){r=parseInt(a,10)}),a.showMeridian=f.showMeridian,b.showMeridian&&a.$parent.$watch(c(b.showMeridian),function(b){if(a.showMeridian=!!b,o.$error.time){var c=g(),d=h();angular.isDefined(c)&&angular.isDefined(d)&&(n.setHours(c),j())}else l()}),this.setupMousewheelEvents=function(b,c){var d=function(a){a.originalEvent&&(a=a.originalEvent);var b=a.wheelDelta?a.wheelDelta:-a.deltaY;return a.detail||b>0};b.bind("mousewheel wheel",function(b){a.$apply(d(b)?a.incrementHours():a.decrementHours()),b.preventDefault()}),c.bind("mousewheel wheel",function(b){a.$apply(d(b)?a.incrementMinutes():a.decrementMinutes()),b.preventDefault()})},this.setupArrowkeyEvents=function(b,c){b.bind("keydown",function(b){38===b.which?(b.preventDefault(),a.incrementHours(),a.$apply()):40===b.which&&(b.preventDefault(),a.decrementHours(),a.$apply())}),c.bind("keydown",function(b){38===b.which?(b.preventDefault(),a.incrementMinutes(),a.$apply()):40===b.which&&(b.preventDefault(),a.decrementMinutes(),a.$apply())})},this.setupInputEvents=function(b,c){if(a.readonlyInput)return a.updateHours=angular.noop,void(a.updateMinutes=angular.noop);var d=function(b,c){o.$setViewValue(null),o.$setValidity("time",!1),angular.isDefined(b)&&(a.invalidHours=b),angular.isDefined(c)&&(a.invalidMinutes=c)};a.updateHours=function(){var a=g();angular.isDefined(a)?(n.setHours(a),j("h")):d(!0)},b.bind("blur",function(){!a.invalidHours&&a.hours<10&&a.$apply(function(){a.hours=i(a.hours)})}),a.updateMinutes=function(){var a=h();angular.isDefined(a)?(n.setMinutes(a),j("m")):d(void 0,!0)},c.bind("blur",function(){!a.invalidMinutes&&a.minutes<10&&a.$apply(function(){a.minutes=i(a.minutes)})})},this.render=function(){var a=o.$viewValue;isNaN(a)?(o.$setValidity("time",!1),d.error('Timepicker directive: "ng-model" value must be a Date object, a number of milliseconds since 01.01.1970 or a string representing an RFC2822 or ISO 8601 date.')):(a&&(n=a),k(),l())},a.showSpinners=angular.isDefined(b.showSpinners)?a.$parent.$eval(b.showSpinners):f.showSpinners,a.incrementHours=function(){m(60*q)},a.decrementHours=function(){m(60*-q)},a.incrementMinutes=function(){m(r)},a.decrementMinutes=function(){m(-r)},a.toggleMeridian=function(){m(720*(n.getHours()<12?1:-1))}}]).directive("timepicker",function(){return{restrict:"EA",require:["timepicker","?^ngModel"],controller:"TimepickerController",replace:!0,scope:{},templateUrl:"template/timepicker/timepicker.html",link:function(a,b,c,d){var e=d[0],f=d[1];f&&e.init(f,b.find("input"))}}}),angular.module("ui.bootstrap.transition",[]).value("$transitionSuppressDeprecated",!1).factory("$transition",["$q","$timeout","$rootScope","$log","$transitionSuppressDeprecated",function(a,b,c,d,e){function f(a){for(var b in a)if(void 0!==h.style[b])return a[b]}e||d.warn("$transition is now deprecated. Use $animate from ngAnimate instead.");var g=function(d,e,f){f=f||{};var h=a.defer(),i=g[f.animation?"animationEndEventName":"transitionEndEventName"],j=function(){c.$apply(function(){d.unbind(i,j),h.resolve(d)})};return i&&d.bind(i,j),b(function(){angular.isString(e)?d.addClass(e):angular.isFunction(e)?e(d):angular.isObject(e)&&d.css(e),i||h.resolve(d)}),h.promise.cancel=function(){i&&d.unbind(i,j),h.reject("Transition cancelled")},h.promise},h=document.createElement("trans"),i={WebkitTransition:"webkitTransitionEnd",MozTransition:"transitionend",OTransition:"oTransitionEnd",transition:"transitionend"},j={WebkitTransition:"webkitAnimationEnd",MozTransition:"animationend",OTransition:"oAnimationEnd",transition:"animationend"};return g.transitionEndEventName=f(i),g.animationEndEventName=f(j),g}]),angular.module("ui.bootstrap.typeahead",["ui.bootstrap.position","ui.bootstrap.bindHtml"]).factory("typeaheadParser",["$parse",function(a){var b=/^\s*([\s\S]+?)(?:\s+as\s+([\s\S]+?))?\s+for\s+(?:([\$\w][\$\w\d]*))\s+in\s+([\s\S]+?)$/;return{parse:function(c){var d=c.match(b);if(!d)throw new Error('Expected typeahead specification in form of "_modelValue_ (as _label_)? for _item_ in _collection_" but got "'+c+'".');return{itemName:d[3],source:a(d[4]),viewMapper:a(d[2]||d[1]),modelMapper:a(d[1])}}}}]).directive("typeahead",["$compile","$parse","$q","$timeout","$document","$window","$rootScope","$position","typeaheadParser",function(a,b,c,d,e,f,g,h,i){var j=[9,13,27,38,40],k=200;return{require:"ngModel",link:function(l,m,n,o){function p(){G.moveInProgress||(G.moveInProgress=!0,G.$digest()),N&&d.cancel(N),N=d(function(){G.matches.length&&q(),G.moveInProgress=!1,G.$digest()},k)}function q(){G.position=B?h.offset(m):h.position(m),G.position.top+=m.prop("offsetHeight")}var r=l.$eval(n.typeaheadMinLength);r||0===r||(r=1);var s,t,u=l.$eval(n.typeaheadWaitMs)||0,v=l.$eval(n.typeaheadEditable)!==!1,w=b(n.typeaheadLoading).assign||angular.noop,x=b(n.typeaheadOnSelect),y=angular.isDefined(n.typeaheadSelectOnBlur)?l.$eval(n.typeaheadSelectOnBlur):!1,z=b(n.typeaheadNoResults).assign||angular.noop,A=n.typeaheadInputFormatter?b(n.typeaheadInputFormatter):void 0,B=n.typeaheadAppendToBody?l.$eval(n.typeaheadAppendToBody):!1,C=l.$eval(n.typeaheadFocusFirst)!==!1,D=n.typeaheadSelectOnExact?l.$eval(n.typeaheadSelectOnExact):!1,E=b(n.ngModel).assign,F=i.parse(n.typeahead),G=l.$new();l.$on("$destroy",function(){G.$destroy()});var H="typeahead-"+G.$id+"-"+Math.floor(1e4*Math.random());m.attr({"aria-autocomplete":"list","aria-expanded":!1,"aria-owns":H});var I=angular.element("<div typeahead-popup></div>");I.attr({id:H,matches:"matches",active:"activeIdx",select:"select(activeIdx)","move-in-progress":"moveInProgress",query:"query",position:"position"}),angular.isDefined(n.typeaheadTemplateUrl)&&I.attr("template-url",n.typeaheadTemplateUrl);var J=function(){G.matches=[],G.activeIdx=-1,m.attr("aria-expanded",!1)},K=function(a){return H+"-option-"+a};G.$watch("activeIdx",function(a){0>a?m.removeAttr("aria-activedescendant"):m.attr("aria-activedescendant",K(a))});var L=function(a,b){return G.matches.length>b&&a?a.toUpperCase()===G.matches[b].label.toUpperCase():!1},M=function(a){var b={$viewValue:a};w(l,!0),z(l,!1),c.when(F.source(l,b)).then(function(c){var d=a===o.$viewValue;if(d&&s)if(c&&c.length>0){G.activeIdx=C?0:-1,z(l,!1),G.matches.length=0;for(var e=0;e<c.length;e++)b[F.itemName]=c[e],G.matches.push({id:K(e),label:F.viewMapper(G,b),model:c[e]});G.query=a,q(),m.attr("aria-expanded",!0),D&&1===G.matches.length&&L(a,0)&&G.select(0)}else J(),z(l,!0);d&&w(l,!1)},function(){J(),w(l,!1),z(l,!0)})};B&&(angular.element(f).bind("resize",p),e.find("body").bind("scroll",p));var N;G.moveInProgress=!1,J(),G.query=void 0;var O,P=function(a){O=d(function(){M(a)},u)},Q=function(){O&&d.cancel(O)};o.$parsers.unshift(function(a){return s=!0,0===r||a&&a.length>=r?u>0?(Q(),P(a)):M(a):(w(l,!1),Q(),J()),v?a:a?void o.$setValidity("editable",!1):(o.$setValidity("editable",!0),a)}),o.$formatters.push(function(a){var b,c,d={};return v||o.$setValidity("editable",!0),A?(d.$model=a,A(l,d)):(d[F.itemName]=a,b=F.viewMapper(l,d),d[F.itemName]=void 0,c=F.viewMapper(l,d),b!==c?b:a)}),G.select=function(a){var b,c,e={};t=!0,e[F.itemName]=c=G.matches[a].model,b=F.modelMapper(l,e),E(l,b),o.$setValidity("editable",!0),o.$setValidity("parse",!0),x(l,{$item:c,$model:b,$label:F.viewMapper(l,e)}),J(),d(function(){m[0].focus()},0,!1)},m.bind("keydown",function(a){if(0!==G.matches.length&&-1!==j.indexOf(a.which)){if(-1===G.activeIdx&&(9===a.which||13===a.which))return J(),void G.$digest();a.preventDefault(),40===a.which?(G.activeIdx=(G.activeIdx+1)%G.matches.length,G.$digest()):38===a.which?(G.activeIdx=(G.activeIdx>0?G.activeIdx:G.matches.length)-1,G.$digest()):13===a.which||9===a.which?G.$apply(function(){G.select(G.activeIdx)}):27===a.which&&(a.stopPropagation(),J(),G.$digest())}}),m.bind("blur",function(){y&&G.matches.length&&-1!==G.activeIdx&&!t&&(t=!0,G.$apply(function(){G.select(G.activeIdx)})),s=!1,t=!1});var R=function(a){m[0]!==a.target&&3!==a.which&&0!==G.matches.length&&(J(),g.$$phase||G.$digest())};e.bind("click",R),l.$on("$destroy",function(){e.unbind("click",R),B&&S.remove(),I.remove()});var S=a(I)(G);B?e.find("body").append(S):m.after(S)}}}]).directive("typeaheadPopup",function(){return{restrict:"EA",scope:{matches:"=",query:"=",active:"=",position:"&",moveInProgress:"=",select:"&"},replace:!0,templateUrl:"template/typeahead/typeahead-popup.html",link:function(a,b,c){a.templateUrl=c.templateUrl,a.isOpen=function(){return a.matches.length>0},a.isActive=function(b){return a.active==b},a.selectActive=function(b){a.active=b},a.selectMatch=function(b){a.select({activeIdx:b})}}}}).directive("typeaheadMatch",["$templateRequest","$compile","$parse",function(a,b,c){return{restrict:"EA",scope:{index:"=",match:"=",query:"="},link:function(d,e,f){var g=c(f.templateUrl)(d.$parent)||"template/typeahead/typeahead-match.html";a(g).then(function(a){b(a.trim())(d,function(a){e.replaceWith(a)})})}}}]).filter("typeaheadHighlight",function(){function a(a){return a.replace(/([.?*+^$[\]\\(){}|-])/g,"\\$1")}return function(b,c){return c?(""+b).replace(new RegExp(a(c),"gi"),"<strong>$&</strong>"):b}}),!angular.$$csp()&&angular.element(document).find("head").prepend('<style type="text/css">.ng-animate.item:not(.left):not(.right){-webkit-transition:0s ease-in-out left;transition:0s ease-in-out left}</style>');

