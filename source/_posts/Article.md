---
extends: _layouts.post
section: content
title: Résolution erreur SQLSTATE [42S22]
date: 2020-03-13
description: Erreur SQLSTATE [42S22]
cover_image: /assets/img/codeur.jpeg
featured: true
---

Bonjour,

Vous voulez ajouter un filtre dans votre vue laravel en utilisant le framework full-stack de Laravel, [Livewire](https://laravel-livewire.com/).
Cependant, il y a un problème:  
# SQLSTATE[42S22]: Column not found: 1054 Unknown column '1' in 'order clause' (SQL: select * from `item` order by `1` asc limit 10 offset 0)

Par la commande <code>php artisan make:livewire WalletTable</code>.

L'exécution de cette commande générera les deux fichiers suivants: 
**WalletTable.php** le component et la vue **wallet-table.blade.php**

```php
namespace App\Http\Livewire;                               //App\Http\Livewire\WalletTable.php

use Livewire\Component;
use Livewire\WithPagination;

class WalletTable extends Component
{
    use WithPagination;

    public $perPage = 10;

    public $sortField;

    public $sortAsc = true;

    public function sortBy($field){

        if ($this->sortField === $field)        // if active field
        {
            $this->sortAsc = ! $this->sortAsc;  // reverse the sort direction
        }else {                                 //else
            $this->sortAsc = true;              //set the direction to true
        }

        $this->sortField = $field;

    }

    public function render()
    {
        return view('livewire.wallet-table', [
           'wallets' => \App\Wallet::query()
                                ->orderBy($this->sortField, $this->sortAsc ? 'asc' : 'desc')
                                ->paginate($this->perPage),
        ]);
    }
}

```
Ce fichier contient déjà la fonction [sort By()] qui permet de filter sur un champ de manière croissante ou décroissante. Et bien évidemment dans le render() la requête sur la collection.

---

```php
<div>                                               //resources/views/livewire/wallet-table.blade.php
<div class="row mb-4">
        <div class="col form-inline">
            Per Page: &nbsp;
            <select wire:model="perPage" class="form-control">
                <option>5</option>
                <option>10</option>
                <option>15</option>
                <option>25</option>
            </select>
        </div>

        <div class="col">
            <input class="form-control" wire:model="search" type="text" placeholder="Search...">
           {{--  <button wire:click="clear">Clear</button> --}}
            <hr>
        </div>
    </div>
    <div class="row">
      <div class="table-responsive">
        <table class="table table-striped table-sm">
          <thead>
            <tr>
              <th><a wire:click.prevent ="sortBy('date')" role="button" href="#"><span data-feather="calendar"></span>
                    Date
                    @include('includes._sort-icon', ['field' => 'date'])
                </a>
              </th>
              <th><a wire:click.prevent ="sortBy('tags')" role="button" href="#"><span data-feather="tag"></span>
                    Tags
                    @include('includes._sort-icon', ['field' => 'tags'])
                </a>
              </th>
              <th><a wire:click.prevent ="sortBy('bank_name')" role="button" href="#"><span data-feather="play-circle"></span>
                    Account
                    @include('includes._sort-icon', ['field' => 'bank_name'])
                </a>
              </th>
              <th><a wire:click.prevent ="sortBy('file')" role="button" href="#"><span data-feather="file"></span>
                    Files
                    @include('includes._sort-icon', ['field' => 'file'])
                </a>
              </th>
              <th><a wire:click.prevent ="sortBy('amount')" role="button" href="#"><span data-feather="hash"></span>
                    Amount
                    @include('includes._sort-icon', ['field' => 'amount'])
                </a>
              </th>
              <th><a wire:click.prevent ="sortBy('type')" role="button" href="#"><span data-feather="shield"></span>
                    Type
                    @include('includes._sort-icon', ['field' => 'type'])
                </a>
              </th>
              <th><a wire:click.prevent ="sortBy('company_name')" role="button" href="#"><span data-feather="file-minus"></span>
                    Organisation
                    @include('includes._sort-icon', ['field' => 'company_name'])
                </a>
              </th>
            </tr>
          </thead>
          <tbody>
            @foreach ($wallets as $wallet)
                <tr>
                    <td>{{ $wallet->date }}</td>
                    <td>{{ $wallet->tags }}</td>
                    <td>{{ $wallet->banque->bank_name }}</td>
                    <td>{{ $wallet->file }}</td>
                    <td>{{ $wallet->amount }}</td>
                    <td>{{ $wallet->type }}</td>
                    <td>{{ $wallet->company->company_name }}</td>
                </tr>
            @endforeach
          </tbody>
        </table>
      </div>

      <div class-="row">
        <div class="col">
            {{ $wallets->links() }}
        </div>
        <div class="col text-right text-muted">
           Affichage de {{ $wallets->firstItem() }} à {{ $wallets->lastItem() }} sur {{ $wallets->total() }} résultats
        </div>
      </div>
    </div>

</div>

```
Cette vue affiche dans un tableau les données contenues dans la table Wallet avec plusieurs autres fonctionnalités dont la pagination.
Un fichier _sort-icon a été inclus pour afficher des icônes montrant un filtre croissant ou un filtre décroissant. il se présente comme suit;

```php
    @if ($sortField !== $field)
    <i class="text-muted fas fa-sort"></i>
    @elseif ($sortAsc)
        <i class="fas fa-sort-up"></i>
    @else
        <i class="fas fa-sort-down"></i>
    @endif

```

Apres un <code>php artisan serve</code> on constate oooopss

 <img src="/assets/img/Sqlstate.png" alt="Error image">


Pour résoudre cette erreur nous devrions specifier dans la requête sur quels champs le filtre sera appliqué pour la première fois. Ceci sera fait dans notre component ici nommé WalletTable.php . C'est cet élément qu'il ne retrouve pas.


```php
namespace App\Http\Livewire;

use Livewire\Component;
use Livewire\WithPagination;

class WalletTable extends Component
{
    use WithPagination;

    public $perPage = 10;

    public $sortField = "date";

    public $sortAsc = true;

    public function sortBy($field){

        if ($this->sortField === $field)
        {
            $this->sortAsc = ! $this->sortAsc;
        }else {
            $this->sortAsc = true;
        }

        // if active field
        // reverse the sort direction
        //else
        //set the direction to true
        $this->sortField = $field;

    }

    public function render()
    {
        return view('livewire.wallet-table', [
           'wallets' => \App\Wallet::query()
                                ->orderBy($this->sortField, $this->sortAsc ? 'asc' : 'desc')
                                ->paginate($this->perPage),
        ]);
    }
}

```

Problème résolu ....

 <img src="/assets/img/success.png" alt="Success image">


Merci pour l'attention. Abonnez-vous a notre site. A la prochaine pour un nouvel article.