ユーザーが賃貸物件を検索するとき、検索対象を特定の都市に限定に限定することもあるでしょう。 特定の都市で検索したり、入力中に検索候補を出すコンポーネントを作成しましょう。

`filter-listing`という名称の新しいコンポーネントを作成しまう。.

```shell
ember g component filter-listing
```

以前と同じように、このコマンドはHandlebars template (`app/templates/components/filter-listing.hbs`) と JavaScript ファイル (`app/components/filter-listing.js`を作成します。).

Handlebars template はこのようになります。

```app/templates/components/filter-listing.hbs City: {{input value=filter key-up=(action 'autoComplete')}} <button {{action 'search'}}>Search</button>

{{#each filteredList as |item|}} <li {{action 'choose' item.city}}>{{item.city}}</li> {{/each}} 

    それには [`{{input}}`](../../templates/input-helpers) helper が含まれていて、ユーザーが入力すると、都市のリストをがフィルタされ、検索することのできるテキストフィールドを描画します。 `input` の`value` property はコンポーネントの`filter` property にバインドされます。
    `key-up` property は`autoComplete` アクションにバインドされます。
    
    また、component (コンポーネント)内の`search` アクションにバインドされるボタンも含んでいます。
    
    最後に、`city` プロパティを表示する、component (コンポーネント)内の各`filteredList` プロパティーのを含んだ順不同のリストが含まれています。 リスト アイテムをクリックすると、その都市の名前アイテムをパラメーターとして 'input' のフィールドに入力する、 `city`プロパティの`choose`アクションが発生します。
    
    コンポーネントのJavaScript は次のようになっています:
    
    ```app/components/filter-listing.js
    export default Ember.Component.extend({
      filter: null,
      filteredList: null,
      actions: {
        autoComplete() {
          this.get('autoComplete')(this.get('filter'));
        },
        search() {
          this.get('search')(this.get('filter'));
        },
        choose(city) {
          this.set('filter', city);
        }
      }
    });
    
    

以上が、上記で説明した各`filter` と`filteredList`とアクションとなります。 興味深いのは、component (コンポーネント)によって定義されているのは`choose` アクションだけです。 各`autoComplete` と`search`アクションのロジックはcomponent's properties (コンポーネントプロパティ)から読み込まれます。つまり、それらのアクションは呼ばれたアクション (../../components/triggering-changes-with-actions/#toc_passing-the-action-to-the-component) を [passed] 引き渡す*closure actions*と呼ばれるデザインパターンです。.

これが、どのように動作するかを確認するために `index.hbs`template (テンプレート)を次のように編集します。

```app/templates/index.hbs 

# Welcome to Super Rentals

We hope you find exactly what you're looking for in a place to stay.   
  
{{filter-listing filteredList=filteredList autoComplete=(action 'autoComplete') search=(action 'search')}} {{#each model as |rentalUnit|}} {{rental-listing rental=rentalUnit}} {{/each}}

{{#link-to 'about'}}About{{/link-to}} {{#link-to 'contact'}}Click here to contact us.{{/link-to}}

    `index.hbs` template (テンプレート)に`filter-listing`component (コンポーネント)を追加しました。 そうして、`filter-listing`component (コンポーネント)が利用するファンクションと、プロパティーを引き渡すことで`index` ページ component (コンポーネント)が、どのように動作するのかを定義することができ、そうすることでcomponent (コンポーネント)はそれらの特定のファンクションとプロパティを利用できるようになります。
    
    これが動作するためにはアプリケーションに `コントローラー` を追加する必要があります、 <0>index. hbs</0>用のコントローラーを自動生成するには
    次のコマンドを実行します。
    
    ```shell
    ember g controller index
    

次のように新しいコントローラを定義します。

```app/controllers/index.js import Ember from 'ember';

export default Ember.Controller.extend({ filteredList: null, actions: { autoComplete(param) { if (param !== '') { this.get('store').query('rental', { city: param }).then((result) => this.set('filteredList', result)); } else { this.set('filteredList', null); } }, search(param) { if (param !== '') { this.store.query('rental', { city: param }).then((result) => this.set('model', result)); } else { this.get('store').findAll('rental').then((result) => this.set('model', result)); } } } });

    <br />以上で見たように、`autoComplete`アクションが参照する`filteredList`controller (コントローラ)のプロパティを定義しました。
     ユーザーがテキストフィールドに入力を行っているとき、このアクションが呼び出されます。 このアクションがレコードの`rental`を参照して、ユーザーがそれまでに入力したものでフィルター処理をします。 このアクションが実行されると、クエーリーの結果は `filteredList`プロパティに置かれ、component (コンポーネント)のオートコンプリートとして用いられます。
    
    また、ここでは`search`アクションを定義して、ボタンがクリックされた時に component (コンポーネント)に引き渡します。 これは、クエーリーの結果が`index`ルートの`モデル`を更新しそれが、ページの賃貸物件のリストを更新するのとは異なります。
    
    これらのアクションが動くにはMirage の`config.js`をリクエストに応えるように、次のように変更する必要があります。
    
    ```mirage/config.js
    export default function() {
      this.get('/rentals', function(db, request) {
        let rentals = [{
            type: 'rentals',
            id: 1,
            attributes: {
              title: 'Grand Old Mansion',
              owner: 'Veruca Salt',
              city: 'San Francisco',
              type: 'Estate',
              bedrooms: 15,
              image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg'
            }
          }, {
            type: 'rentals',
            id: 2,
            attributes: {
              title: 'Urban Living',
              owner: 'Mike Teavee',
              city: 'Seattle',
              type: 'Condo',
              bedrooms: 1,
              image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg'
            }
          }, {
            type: 'rentals',
            id: 3,
            attributes: {
              title: 'Downtown Charm',
              owner: 'Violet Beauregarde',
              city: 'Portland',
              type: 'Apartment',
              bedrooms: 3,
              image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg'
            }
          }];
    
        if(request.queryParams.city !== undefined) {
          let filteredRentals = rentals.filter(function(i) {
            return i.attributes.city.toLowerCase().indexOf(request.queryParams.city.toLowerCase()) !== -1;
          });
          return { data: filteredRentals };
        } else {
          return { data: rentals };
        }
      });
    }
    

これらの変更で、ユーザーは、入力候補が出る検索フィールドで好みの都市で賃貸不動産を検索することができます。