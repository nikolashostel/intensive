if (!window["App"]) throw new Error("Critical Error in Application");
(function m(App) {


    //===========================================================================================================


    function lib() {
        var _windowContainer = null,
        _guid = null,
        _poll = this,
        _name = '',
        _store = null;

        var header, panel;

        var save = function(){ //Функция сохранения изменений в анкете
            if (!panel.container['form'].pharmId) return false;

            var modifyData = '';

            $.each(_store.getModifiedRecords(), function(index, rec){
                $.each(rec.modified, function(name){
                    modifyData += '&values['+index+']['+name+']=' + rec.data[name];
                });
                modifyData += '&id['+index+']=' + rec.id;
            });
            if (modifyData == '') return;
                $.getJSON('./modules/units/units.php?action=SET_FORMDATA&pharmId='+panel.container['form'].pharmId+'&projectId='+_param.projectId+'&stageId='+_param.stageId + modifyData +'&rnd='+Math.random(), function(data){
                if (!data || !data.result){
                    alert('ошибка сохранения, попробуйте еще раз');
                    //_store.rejectChanges();
                }else if (data.result == true){
                    _store.commitChanges();
                    panel.container['form'].loadData(panel.container['form'].pharmId);
                }
                $.unblockUI();
            }.bind(this));
        }

        var Header = function (container) {
            var id = 'li[rel = "' + _guid + '"] '+ ".context";
            var template = '<div class="header" style="clear:both; font-size: small;"><div class="context" style="clear:both; overflow:hidden"></div><div class="header_button collapse"></div><div class="buttons">&nbsp;</div></div>';
            Header.superClass.apply(this, [id, template, container]);

            this.appendTo = function (parent) {
                $(parent).append(this.template);
                return this;
            }

            var _expanded = true;

            this.expand = function(callback){
                _expanded = true;
                _windowContainer.find('.header_button').removeClass('expand').addClass('collapse');
                this.element().animate({
                    height: this.container['title'].element().outerHeight() + this.container['description'].element().outerHeight()
                }, "fast", callback);
            }

            this.height = function (height) {
                return _windowContainer.find('.header').outerHeight()
            }

            this.expanded = function(){
                return _expanded;
            }

            this.addButton = function(caption, delegate){
                _windowContainer.find('.buttons').append('<input type="button" id="'+caption+'" value="'+caption+'" />');
                _windowContainer.find('.buttons #'+caption).click(delegate);
            }

            this.collapse = function(callback){
                _expanded = false;
                _windowContainer.find('.header_button').removeClass('collapse').addClass('expand');
                this.element().animate({
                    height: this.container['title'].element().outerHeight()
                }, "fast", callback);
            }

            this.toggle = function(callback){
                if (_expanded){
                    this.collapse(callback);
                }else{
                    this.expand(callback);
                }
            }

            this.click = function(delegate){
                if (!delegate){
                    _windowContainer.find('.header_button').click();
                }else{
                    _windowContainer.find('.header_button').unbind().click(delegate);
                }
            }
        }
        Header.inherits(App.VisualElement);//Наследуем элемент от базового из файла part_of_core.js

        var Title = function(){
            var id = 'li[rel = "' + _guid + '"] '+ ".title";
            var template = '<div class="title" style="clear:both">Приближая весну</div>';
            Title.superClass.apply(this, [id, template]);
        }
        Title.inherits(App.VisualElement);
        
        var Description = function(){
            var id = 'li[rel = "' + _guid + '"] '+ ".description";
            var template = '<div class="description" style="clear:both"></div>';
            Description.superClass.apply(this, [id, template]);
        }
        Description.inherits(App.VisualElement);
                
        var Panel = function(container){
            var id = 'li[rel = "' + _guid + '"] '+ ".panel";
            var template = '<div class="panel" style="width: 100%;"></div>';
            Panel.superClass.apply(this, [id, template, container]);
        }
        Panel.inherits(App.VisualElement);

        var Grid = function () { 
            var id = 'li[rel = "' + _guid + '"] '+ ".grid";
            var template = '<div class="grid" style="background-color: #98fb98; float: left; width: 250px;"></div>';
            Grid.superClass.apply(this, [id, template]);

            var store = new Ext.data.JsonStore({// аналог "DataTable" из состава ExtJS
                root: 'units',
                totalProperty: 'count',
                idProperty: 'id',
                remoteSort: false,

                fields: [
                    'id', 'name', 'address', 'town', {name: 'checked', type: 'bool'}
                ],

                url: 'modules/units/units.php?action=GET_ACTIVE_PHARMACIES&projectId='+_param.projectId+'&stageId='+_param.stageId+'&rnd='+Math.random()
            });
            store.setDefaultSort('id', 'desc');

            function renderTopic(value, p, record) {
                return String.format('{1}, {2} {3}', record.data.town, value, record.data.address);
            }

            var grid = new Ext.grid.GridPanel({//многофункциональный грид из состава ExtJS
                store: store,
                trackMouseOver: false,
                disableSelection: false,
                width: 250,
                loadMask: new Ext.LoadMask(Ext.getBody(), {onLoad: function(){ $.unblockUI(); } , onBeforeLoad: function(){ $.blockUI(); } }),

                columns: [
                {
                    header: "Аптека",
                    dataIndex: 'name',
                    width: 200,
                    renderer: renderTopic,
                    sortable: true
                }],

                viewConfig: {
                    forceFit: true,
                    enableRowBody: true,
                    showPreview: true
                }
            });

            this.rowClick = function(event){
                grid.on('rowclick', function(grid, rowIndex, e) {
                    var r = grid.getStore().getAt(rowIndex);
                    event(r, rowIndex);
	            });
            }            

            this.height = function (height) {
                grid.setHeight(height);
            }

            this.appendTo = function (parent) {
                $(parent).append(this.template);

                store.load({
                    params: { rnd: Math.random() },
                    callback: function () {
                        $(_poll).trigger({ type: "poll", event: "complete" });
                    } .bind(this)
                });

                grid.render(Ext.select(this.id, true));
            }
        }
        Grid.inherits(App.VisualElement);

        var Form = function(){
            var id = 'li[rel = "' + _guid + '"] '+ ".form";
            var template = '<div class="form" style="margin: 0 auto;"></div>';
            Form.superClass.apply(this, [id, template]);
            
            var grid, height;

            this.pharmId = null;

            var config = {
                forceFit: true,
                enableRowBody: false
            };

            this.appendTo = function (parent) {
                $(parent).append(this.template);
                loadHeader.call(this);
            }

            this.height = function (newHeight) {
                height = newHeight;
                if (grid){
                    grid.setHeight(height);
                }
            }

            var loadHeader = function(){ //Шапка анкеты приходит из базы, но только один раз
                $.getJSON('./modules/units/units.php?action=GET_FORMHEADER&projectId=' + _param.projectId + '&stageId=' + _param.stageId + '&rnd='+Math.random(), function(data){
                    var mask = new Ext.LoadMask(Ext.getBody(), {onLoad: function(){ $.unblockUI(); } , onBeforeLoad: function(){ $.blockUI(); } });                    

                    _store = new Ext.data.GroupingStore({
                        reader: new Ext.data.JsonReader({fields: Ext.data.Record.create(data.fields)})
                    });

                    if ($.isArray(data.groups)){
                        var group = new Ext.ux.grid.ColumnHeaderGroup({
                            rows: data.groups
                        });
                    }

                    var proxy = new Ext.data.HttpProxy(
			        {
			            url: './modules/units/units.php?action=GET_LIST&rnd'+Math.random(),
			            method: 'POST'
			        });                  

                    for (var i = 0; i < data.columns.length; i++){
                        if (data.columns[i].editor && data.columns[i].editor.xtype == 'combo'){
                            data.columns[i].width = 130;
                            var store = new Ext.data.JsonStore(
                            {
                                fields: ['rowId','name'],
                                proxy: proxy,
                                baseParams: { listId: data.columns[i].editor.listId },
                                idProperty: 'rowId',
                                root: 'records',
                                autoLoad: true
                            });
                            data.columns[i].editor =  { 
                                xtype: 'combo',
                                typeAhead: true,
                                lazyRender: true,
                                triggerAction: 'all',
                                store: store,
                                mode: 'local', 
                                displayField: 'name',
                                valueField: 'rowId',
                                editable: false,
                                selectOnFocus:true
                            }
                        }
                    }

                    grid = new Ext.grid.EditorGridPanel({ //Грид анкеты
                        store: _store,
                        columns: data.columns,
                        width: 'auto',
                        height: height,
                        loadMask: mask,
                        clicksToEdit: 0,
                        viewConfig: config,
                        plugins: ($.isArray(data.groups)? group : undefined)
                    });

                    grid.render(Ext.select(this.id, true));
                }.bind(this));
            }

            this.loadData = function(pharmId){ //Загрузка данных анкеты в созданую таблицу (Form)
                this.pharmId = pharmId;
                _store.rejectChanges();
                $.getJSON('./modules/units/units.php?action=GET_FORMDATA&pharmId=' + pharmId + '&projectId=' + _param.projectId + '&stageId=' + _param.stageId + '&rnd='+Math.random(), function(data){
                    _store.loadData(data.rows);
                    $.unblockUI();
                });
            }
        }
        Form.inherits(App.VisualElement);

        //===========================================================================================================


        function main(viewPort) { //создается окно модуля
            _windowContainer = viewPort;

            header = new Header(); //шапка модуля (документа)
            panel = new Panel();

            header.appendTo(_windowContainer.find('.body')); //добавляем ее на рабочую область
            header.add({
                title: new Title(),
                description: new Description()
            }); //Добавляем на шапку рабочие элементы

            header.container['description'].element().load('./modules/units/forms.html?'+Math.random(), function(){
                $.unblockUI();
                panel.container['grid'].height($('#window').outerHeight() - header.height());
                panel.container['form'].height($('#window').outerHeight() - header.height());
            }.bind(this)); //загружаем "статический" текст-описание

            header.addButton('save', function(){
                save();
            }); //обработчик на кнопку сохранения

            panel.appendTo(_windowContainer.find('.body')); //Панель эта главная рабочая область окна
            panel.add({
                grid: new Grid(),
                form: new Form()
            }); //На нее добавляем список аптек и таблицу - анкету

            panel.container['grid'].rowClick(function(record, index){
                panel.container['form'].loadData(record.id);
            }); //При выборе аптеки подгружаем анкету

            header.click(function(){
                panel.hide();
                header.toggle(function(){
                    panel.show();                    
                    panel.container['grid'].height($('#window').outerHeight() - header.height());
                    panel.container['form'].height($('#window').outerHeight() - header.height());
                });
            }); //При нажатии на шапку - сворачиваем ее
        }

        var sync = function (message) {
            switch (message.event) {//сообщения приходят только от ядра, 
                                    //если другой модуль хочет отправить сообщение этому модулю
                                    //то все равно его "прокидывает" ядро по таргету
                case "binding": //сообщение приходит один раз при успешной привязке к ядру
                    $(window.document).unbind('ajaxStart');
                    _guid = message.guid;
                    _name = message.name;
                    _param = message.data;
                    if (_param.edit == undefined) {
                        _param.edit = true
                    }
                    main(message.viewPort);
                    break;
                case "unbinding": //сообщение приходит один раз при выгрузке модуля
                    $(window.document).ajaxStart($.blockUI);
                    break;
                case "show": //сообщение приходит всякий раз когда модуль становится "активным", 
                             //т.е. окно модуля получает фокус
                    $(window.document).unbind('ajaxStart');
                    break;
                case "hide": //сообщение приходит при скрытии окна 
                    $(window.document).ajaxStart($.blockUI);
                    break;
                case "message": //любое другое сообщение, например служебное или от другого модуля
                    alert(message.body.test);
                    break;
            }
        };

        this.entry = function (message) {
            sync(message); //Точка входа в модуль
        };

        this.getGuid = function () { //каждый экземпляр модуля получает уникальный id, 
                                     //на один модуль может быть много экземпляров - в аналогии с ООП
                                     //модуль это класс, экземпляр модуля это объект
            return _guid;
        };

        this.getName = function () {
            return _name;
        };
    }


    //===========================================================================================================

    App.Core.Registry(lib, "units_forms"); //Регистрация модуля в ядре
})(App);
